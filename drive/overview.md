# Drive surface overview

gws exposes Drive two ways:

1. **Helpers** — today only `gws drive +upload`
2. **Raw API** — `gws drive <resource> <method> --params '…' [--json '…']`

Always invoke as `gws drive …` (not bare `gws files` / `gws permissions`). Help
text inside the Drive tree often omits the `drive` prefix.

Discover shapes anytime:

```bash
gws drive --help
gws drive files --help
gws drive permissions --help
gws schema drive.files.list
gws schema drive.permissions.create --resolve-refs
```

## Topic map

| Topic | File |
|-------|------|
| Share files, folders, drives | [sharing.md](./sharing.md) |
| My Drive vs shared drives vs consumer | [my-drive-vs-shared-drives.md](./my-drive-vs-shared-drives.md) |

## Resource tree (raw)

```text
drive
  +upload
  about            get
  files            list get create update copy delete download export
                   emptyTrash generateIds watch listLabels modifyLabels
  permissions      list get create update delete
  drives           list get create update delete hide unhide
  accessproposals  list get resolve
  approvals        list get start approve decline cancel comment reassign
  comments         …
  replies          …
  revisions        …
  changes          …
  channels         …
  apps             …
  operations       …
  teamdrives       deprecated alias of drives
```

Folders are files with MIME type `application/vnd.google-apps.folder`. The same
`files` and `permissions` methods apply to files and folders.

## Common create / list

```bash
# folder in My Drive root
gws drive files create \
  --json '{"name":"NAME","mimeType":"application/vnd.google-apps.folder"}'

# small local file into a folder
gws drive +upload ./note.txt --parent FOLDER_ID --name 'note.txt'

# children of a folder
gws drive files list --params '{
  "q":"'\''FOLDER_ID'\'' in parents and trashed=false",
  "fields":"files(id,name,mimeType,shared)",
  "pageSize":20
}'
```

`+upload` detects MIME type and filename. Raw `files.create` is required for
folders (no bytes) and for anything that needs extra metadata.

## Safety (Drive-specific)

- Read-only (`list` / `get` / `about.get`) unless the user asks to mutate.
- Prefer `files.update` with `"trashed": true` over `files.delete` (permanent).
- Trashing a folder eventually marks children `trashed: true` with
  `explicitlyTrashed: false`. Re-read after a few seconds; the first get can
  still show `trashed: false`.
- `files.delete` on a folder permanently deletes descendants the user owns.
- Do not create `type=anyone` permissions on real data while exploring.
- `permissions.create` emails the grantee by default (`sendNotificationEmail`
  defaults true for user/group).
- Concurrent permission writes on the same item are not supported; last write
  wins.

## Scopes

Mutating permissions needs `https://www.googleapis.com/auth/drive` or
`drive.file`. `drive.file` only covers files the app created or the user
explicitly opened with the app. Shared-drive calls need
`supportsAllDrives: true` (and usually `includeItemsFromAllDrives` on
`files.list`).
