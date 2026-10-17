# Share files, folders, and drives

Sharing is the Drive **permissions** resource. There is no `+share` helper.

```bash
gws drive permissions list|get|create|update|delete \
  --params '{…}' [--json '{…}']
```

Official guide: [Share files, folders, and drives](https://developers.google.com/workspace/drive/api/guides/manage-sharing).
Roles: [Roles and permissions](https://developers.google.com/workspace/drive/api/guides/ref-roles).

My Drive vs shared drives (and consumer vs commercial Workspace) is a different
ownership model, not a different gws command: [my-drive-vs-shared-drives.md](./my-drive-vs-shared-drives.md).

## What a permission is

Each permission is a grantee `type` plus a `role`.

| `type` | Also send | Meaning |
|--------|-----------|---------|
| `user` | `emailAddress` | one Google account |
| `group` | `emailAddress` | a Google Group |
| `domain` | `domain` | everyone in that Workspace domain |
| `anyone` | (none) | anyone with the link (`allowFileDiscovery` controls search) |

| `role` | Drive UI (My Drive) | Drive UI (shared drive) |
|--------|---------------------|-------------------------|
| `owner` | Owner | *not used* |
| `organizer` | — | Manager |
| `fileOrganizer` | — | Content manager |
| `writer` | Editor | Contributor |
| `commenter` | Commenter | Commenter |
| `reader` | Viewer | Viewer |

`organizer` and `fileOrganizer` are shared-drive only. Creating either on a My
Drive item fails with `organizerOnNonTeamDriveNotSupported` /
`fileOrganizerOnNonTeamDriveNotSupported`.

The link does not change per person. Access is the item's ACL evaluated against
the signed-in user. `webViewLink` on `files.get` is the URL to hand out.

## Fields are not optional if you care

Discovery does not list `fields`, but gws accepts it inside `--params`. Default
`permissions.list` returns only `id`, `kind`, `type`, `role`. No email, no
inheritance, no `allowFileDiscovery`.

```bash
gws drive permissions list --params '{
  "fileId":"FILE_ID",
  "fields":"permissions(id,type,role,emailAddress,displayName,domain,allowFileDiscovery,expirationTime,permissionDetails,deleted)"
}'
```

On My Drive, `files.get` with `fields=permissions,…` also returns the ACL
(including inherited rows). On shared-drive items, `files.permissions` is
**not** populated — always `permissions.list` (paginated; default page size 100).

`permissionDetails[]` is how you see inheritance:

```json
{
  "id": "anyoneWithLink",
  "type": "anyone",
  "role": "commenter",
  "permissionDetails": [
    { "inherited": true, "permissionType": "file", "role": "commenter" }
  ]
}
```

The owner on a My Drive item typically shows two details: inherited `writer`
from a parent, plus a direct `owner` row.

## Create / update / delete

```bash
# user (notification email ON by default)
gws drive permissions create \
  --params '{"fileId":"FILE_ID","sendNotificationEmail":true}' \
  --json '{"type":"user","role":"writer","emailAddress":"user@example.com"}'

# anyone-with-link, view only, not searchable
gws drive permissions create \
  --params '{"fileId":"FILE_ID"}' \
  --json '{"type":"anyone","role":"reader","allowFileDiscovery":false}'

# change role
gws drive permissions update \
  --params '{"fileId":"FILE_ID","permissionId":"PERMISSION_ID"}' \
  --json '{"role":"commenter"}'

# revoke
gws drive permissions delete \
  --params '{"fileId":"FILE_ID","permissionId":"PERMISSION_ID"}'
```

The anyone-with-link permission id is the literal string `anyoneWithLink`.

Create always needs `type` + `role`. For `user`/`group` also send
`emailAddress` even though the discovery schema marks that field read-only.

Useful query flags on create:

| Param | Notes |
|-------|--------|
| `sendNotificationEmail` | Defaults **true** for user/group. Cannot be false on an ownership transfer. |
| `emailMessage` | Plain-text note in that email. |
| `transferOwnership` | Required acknowledgement when making someone else `owner`. |
| `moveToNewOwnersRoot` | My Drive only; with transfer, move the item to the new owner's root. |
| `supportsAllDrives` | Set true whenever the item might live on a shared drive. |

### Inviting an address that is not a Google account

`sendNotificationEmail: false` fails with `invalidSharingRequest`: Drive will
not create the permission unless it can notify that address. Only turn notify
off for addresses that already have a Google account.

## Inheritance (how a “directory share” works)

Permission lists propagate down the folder tree. Sharing a folder is how you
share a directory. There is no separate recursive share-every-file call.

- Inherited permissions **cannot** be deleted or reduced on the child.
  `permissions.delete` on an inherited id returns `cannotDeletePermission` and
  points at [limited access](https://developers.google.com/workspace/drive/api/guides/limited-expansive-access).
- They **can** be increased on the child. A later parent change does not lower
  that child grant unless the parent becomes *more* permissive than the child.
- Moving a file into a new folder applies that folder's ACL (and drops the old
  folder's inherited roles). Direct grants on the file stay.
- After a share, role change, revoke, or folder trash, re-read. The first
  `permissions.list` / `files.get` on a child can lag a few seconds (missing
  inherited row, stale `role`, or `trashed: false`).

### Limited access

Set `inheritedPermissionsDisabled: true` on a **folder** (`files.update`). Only
the My Drive `owner` (or shared-drive `organizer`) and people granted **directly
on that folder** can open it. Parent-only grants still see the folder name but
not its contents.

```bash
gws drive files update \
  --params '{"fileId":"FOLDER_ID","fields":"id,inheritedPermissionsDisabled"}' \
  --json '{"inheritedPermissionsDisabled":true}'
```

Verified on My Drive: after flipping this on a subfolder, the parent
`anyoneWithLink` row can still appear on the subfolder ACL, but `shared`
becomes `false` and children no longer show that anyone grant. Do not treat a
listed inherited row as “this person can open the file.” Check `shared` and
`permissionDetails.inherited`, and prefer `files.capabilities` for what **you**
can do.

Limited access is folders only (`canDisableInheritedPermissions` is false on
plain files).

## What actually decides “can I share this?”

The permissions list is the ACL. What the **current caller** can do is
`files.capabilities` (`canShare`, `canEdit`, `canDelete`,
`canDisableInheritedPermissions`, …). Apps should key UI off capabilities, not
re-derive them from role names.

My Drive share rules (docs):

- File or folder: caller needs `writer` or `owner`.
- If `writersCanShare` is false, only `owner`.
- A writer whose access is time-limited cannot share.
- Folder expirations are only allowed on `role=reader`.

Shared-drive share rules differ — see [my-drive-vs-shared-drives.md](./my-drive-vs-shared-drives.md).

## Nearby resources

| Resource | Use |
|----------|-----|
| `accessproposals` | Pending “request access” on a file. `list`/`get`/`resolve`. |
| `approvals` | Review workflow on a file (`start`, `approve`, `decline`, …). |

On an unshared My Drive folder these return empty wrappers (`{}` /
`drive#approvalList`). Not the same as permissions.

## gws footguns

- **Empty DELETE body.** `permissions.delete` succeeds with an empty body. gws
  0.22.x treats that as a binary download and writes `download.html` (0 bytes)
  in the cwd. Delete the leftover; the API call still worked. Prefer running
  deletes from a scratch directory, or pass `-o` to a throwaway path.
- **Default field set is incomplete.** Always set `fields` when auditing who
  has access.
- **Schema vs create body.** `emailAddress` / `domain` look output-only in
  `gws schema` and are still required on create for that `type`.
- **`shared: false` after create.** A brand-new My Drive file is unshared until
  a non-owner grant exists. Folder-level `anyone` makes children `shared: true`
  once inheritance lands.
- **Do not use `files.delete` to unshare.** That deletes the file.
