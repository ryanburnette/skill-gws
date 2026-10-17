# Gmail API map (via gws)

Invocation: `gws gmail users <resource> <method> --params '{…}' [--json '{…}']`  
Helpers: `gws gmail +name …`  
Schema: `gws schema gmail.users.messages.list` or `gws schema gmail.Message`

## Helpers

| Command | Risk | Notes |
|---------|------|--------|
| `+triage` | read | Default `q=is:unread`; table unless `--format json` |
| `+read` | read | Decodes body; `--headers`; `--html` |
| `+send` | send / draft | `--draft`, `--attach`, `--from`, `--html` |
| `+reply` / `+reply-all` | send / draft | Threading + quote |
| `+forward` | send / draft | Keeps attachments by default |
| `+watch` | setup + read stream | Pub/Sub; renew ~7d |

## users

| Method | Risk |
|--------|------|
| `getProfile` | read |
| `watch` / `stop` | push config |

## messages

| Method | Risk | Notes |
|--------|------|--------|
| `list` | read | `q`, `labelIds` (AND), `includeSpamTrash`, maxResults ≤500 |
| `get` | read | format: minimal \| metadata \| full \| raw |
| `modify` | mutate | labels; up to 100 add + 100 remove |
| `batchModify` | mutate | up to **1000** ids; empty success body |
| `trash` / `untrash` | recoverable | prefer over delete |
| `send` | send | prefer `+send` |
| `import` / `insert` | mailbox write | not external send |
| `delete` / `batchDelete` | **permanent** | avoid |
| `attachments.get` | read | base64url `data` |

## threads

| Method | Risk | Notes |
|--------|------|--------|
| `list` / `get` | read | get can return many messages |
| `modify` | mutate | all messages in thread |
| `trash` / `untrash` | recoverable | whole thread |
| `delete` | **permanent** | whole thread |

## labels

| Method | Risk | Notes |
|--------|------|--------|
| `list` | read | no counts |
| `get` | read | includes totals |
| `create` / `update` / `patch` | mutate | user labels |
| `delete` | mutate | strips label from mail |

## drafts

| Method | Risk |
|--------|------|
| `list` / `get` | read |
| `create` / `update` | mutate |
| `send` | send |
| `delete` | permanent draft remove |

## history

| Method | Risk | Notes |
|--------|------|--------|
| `list` | read | `startHistoryId` required in practice; stale → 404 |

## settings

| Area | Methods | Risk |
|------|---------|------|
| vacation, imap, pop, language, autoForwarding | get / update | read / account mutate |
| filters | list get create delete | create/delete = rules |
| sendAs | list get create patch update delete verify + smime | identity |
| forwardingAddresses | list get create delete | forwarding |
| delegates / cse | * | often restricted |

## Workflow (cross-service)

| Helper | Needs | Risk |
|--------|-------|------|
| `gws workflow +email-to-task` | Gmail + Tasks | creates task |
| `gws workflow +weekly-digest` | Gmail + Calendar | read |

## Pagination flags (gws)

| Flag | Meaning |
|------|---------|
| `--page-all` | Follow `nextPageToken`; NDJSON one page per line |
| `--page-limit N` | Max pages (default 10) |
| `--page-delay MS` | Delay between pages (default 100) |
