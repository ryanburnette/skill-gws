# My Drive vs shared drives vs consumer accounts

gws is the same CLI on every account. Sharing still goes through
`gws drive permissions`. What changes is **who owns the file** and **which
roles exist**.

This page mixes official API rules with what we have actually run. Shared-drive
behavior below is from Google's docs, not from a live shared drive.

## Three layers people conflate

| Layer | What it is |
|-------|------------|
| **Account type** | Consumer Google Account (`@gmail.com` and similar) vs a **Google Workspace** user in an organization. |
| **Storage location** | **My Drive** (user-owned tree) vs a **shared drive** (org-owned tree). |
| **Admin policy** | Workspace-only controls: external sharing, link sharing, default link visibility, target audiences, whether users may create shared drives. |

A Workspace user still has a My Drive. Shared drives sit *beside* it. A file
lives in one or the other, not both (shortcuts can point across).

## Consumer accounts (verified)

`gws drive about get` with `fields=canCreateDrives,user` is the capability
check. Consumer accounts report `canCreateDrives: false`.

```bash
gws drive about get --params '{"fields":"canCreateDrives,user(emailAddress,permissionId)"}'
gws drive drives list --params '{"pageSize":10}'
```

Observed on a consumer My Drive:

| Call | Result |
|------|--------|
| `drives.list` | Empty `drives` array |
| `drives.create` | `403 userCannotCreateTeamDrives` |
| `drives.list` + `useDomainAdminAccess: true` | `400 invalid` |
| `permissions.create` `role=organizer` or `fileOrganizer` | `403` “…only valid for shared drives” |
| `type=domain` `domain=gmail.com` | `400` invalid / not applicable |
| Folder + file share via `permissions.*` | Works (user / anyone) |

So: **gws can share consumer My Drive files and folders.** It cannot create or
manage shared drives there, and domain-wide sharing is a Workspace feature.

## Commercial Workspace

Shared drives are a Workspace feature (Business editions including Starter as of
late 2024, Enterprise, Education, Essentials, Nonprofits — confirm on the
current [admin help](https://support.google.com/a/answer/7337469) if it
matters). The org admin can still turn creation off.

Workspace-only extras that affect sharing:

- `type=domain` for the org's real domain (and [target audiences](https://support.google.com/a/answer/9934697) as `ID.audience.googledomains.com`).
- `useDomainAdminAccess` on `drives.*` / `permissions.*` when a domain admin
  manages a shared drive they are not a member of.
- Admin settings can forbid `anyone` links, external users, or sharing outside
  selected domains. The API then errors even though the gws command is right.
- Drive labels, approvals, and some access-proposal flows show up more often.

gws does not hide any of this. A rejected share is usually policy or role, not a
missing subcommand.

## Ownership and roles

| | My Drive | Shared drive |
|--|----------|--------------|
| Owner of a file | A user (`owners[]`, `ownedByMe`) | The **shared drive**, not a person |
| `owner` role | Yes | Not allowed |
| Membership | N/A (just ACL on items) | Users/groups on the **drive** (`permissions` on the drive id) plus optional extra grants on files/folders |
| Extra roles | — | `organizer` (Manager), `fileOrganizer` (Content manager) |
| `writersCanShare` | Honored | Treated as always true |
| `shared` | Meaningful | Always shared; field not populated |
| `files.permissions` on `files.get` | Returned if you ask | **Omitted** (ACL can be huge). Use `permissions.list` |
| Inheritance | Expansive, plus [limited-access folders](https://developers.google.com/workspace/drive/api/guides/limited-expansive-access) | Same expansive default, but membership is a floor: you cannot drop a member below their drive role except via limited access |
| Parents | One parent | Exactly one parent; item belongs to one shared drive |

Two permission classes on a shared drive ([overview](https://developers.google.com/workspace/drive/api/guides/about-shareddrives)):

1. **Member** — access to the whole drive (and its metadata). Only `user` and
   `group` (not `anyone` / `domain`) can be members.
2. **File** — extra access to a subset of items. A file grant can *raise* a
   member's role on that item; it cannot *lower* it below membership.

Who may share (docs):

- Shared-drive **file**: `writer`, `fileOrganizer`, or `organizer`.
- Shared-drive **folder**: `organizer`, unless the drive restriction
  `sharingFoldersRequiresOrganizerPermission` is false, in which case
  `fileOrganizer` can share folders too.
- Drive membership: `organizer` only.

`permissionDetails` on shared-drive items is the way to see whether a role
comes from membership or from a file/folder grant (`inheritedFrom`,
`permissionType: member|file`).

## API flags you must set for shared drives

Almost every `files` / `permissions` / `changes` call that might touch a shared
drive needs:

```json
{ "supportsAllDrives": true }
```

List files across or inside one:

```bash
# everything the user can see, including shared drives
gws drive files list --params '{
  "corpora":"allDrives",
  "includeItemsFromAllDrives":true,
  "supportsAllDrives":true,
  "pageSize":20,
  "fields":"files(id,name,mimeType,driveId,parents),nextPageToken"
}'

# one shared drive
gws drive files list --params '{
  "corpora":"drive",
  "driveId":"DRIVE_ID",
  "includeItemsFromAllDrives":true,
  "supportsAllDrives":true,
  "pageSize":20
}'
```

Create a shared drive (Workspace, `canCreateDrives: true`):

```bash
gws drive drives create \
  --params '{"requestId":"IDEMPOTENCY-UUID"}' \
  --json '{"name":"Team files"}'
```

`requestId` is required (idempotency). Restrictions are not set on create; use
`drives.update` after.

## Fields that only exist (or vanish) on shared-drive files

From [Shared drive versus My Drive API differences](https://developers.google.com/workspace/drive/api/guides/shared-drives-diffs):

Present only on shared-drive items: `driveId`, `hasAugmentedPermissions`,
several `capabilities.can*Drive*` / `canDeleteChildren` / `canTrashChildren`
flags, `trashedTime`, `trashingUser`.

Not populated: `permissions` (on the file resource), `owners` / `ownedByMe`,
`folderColorRgb`, `shared`, `writersCanShare`.

`parents.isRoot` is true only for **My Drive** root, never for the shared
drive's top folder (that folder's id **is** the drive id).

## What we have not exercised

No commercial Workspace login and no shared drive were available when these
notes were written. Re-verify on a Workspace user before automating:

- `drives.create` / membership `permissions.create` on a drive id
- `supportsAllDrives` on every share path
- `files.get` omitting `permissions`
- membership floor vs file-level upgrade
- admin policy errors vs API misuse
- `useDomainAdminAccess`

Until then, treat this file as the map and [sharing.md](./sharing.md) as the
verified My Drive recipe.
