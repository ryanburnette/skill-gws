# Gmail surface overview

gws exposes Gmail two ways:

1. **Helpers** — `gws gmail +triage`, `+read`, `+send`, `+reply`, `+reply-all`,
   `+forward`, `+watch`
2. **Raw API** — `gws gmail users <resource> <method> --params '…' [--json '…']`

Prefer helpers for human exploration. Prefer raw methods when you need labels,
history, filters, attachments, batch modify, or exact API control.

Always invoke raw methods as `gws gmail users …` (not bare `gws users …`).

Discover shapes anytime:

```bash
gws schema gmail.users.messages.list
gws schema gmail.FilterAction
gws gmail --help
gws gmail users messages --help
```

## Topic map

| Topic | File |
|-------|------|
| List / get / threads / profile | [reading.md](./reading.md) |
| UI tabs vs categories | [categories-and-tabs.md](./categories-and-tabs.md) |
| Archive, read/unread, stars, labels, batch | [organize.md](./organize.md) |
| Send, reply, forward, drafts | [compose.md](./compose.md) |
| Download attachments | [attachments.md](./attachments.md) |
| Incremental sync + Pub/Sub watch | [history-and-watch.md](./history-and-watch.md) |
| Filters, vacation, send-as, forwarding | [settings-and-filters.md](./settings-and-filters.md) |
| Trash vs permanent delete | [destructive-ops.md](./destructive-ops.md) |
| Method risk table | [helpers-and-raw.md](./helpers-and-raw.md) |
| Query recipes | [../reference/gmail-queries.md](../reference/gmail-queries.md) |
| Compact API map | [../reference/gmail-api-map.md](../reference/gmail-api-map.md) |

## Resource tree (raw)

```text
gmail.users
  getProfile | watch | stop
  messages     list get modify batchModify trash untrash
               send import insert delete batchDelete
               attachments.get
  threads      list get modify trash untrash delete
  labels       list get create update patch delete
  drafts       list get create update delete send
  history      list
  settings     get/update Vacation|Imap|Pop|Language|AutoForwarding
               filters | sendAs | forwardingAddresses
               delegates | cse   (often Workspace / special auth)
```

## Cross-service helpers

```bash
gws workflow +email-to-task --message-id ID   # needs Tasks scope; mutates Tasks
gws workflow +weekly-digest                   # calendar week + unread estimate
```

## Safety (Gmail-specific)

- Default to read-only (`list` / `get` / `+triage` / `+read`).
- Mutate only with explicit user intent.
- Prefer **trash** over `delete` / `batchDelete`.
- `gmail.modify` includes **send** — do not treat it as “organize only.”
- Before bulk mutate: paginated real count, metadata sample, confirm query.
- Large mailboxes: tight `q`, bounded `maxResults` (API max **500**/page), no
  casual `--page-all`.
- Server-side **filters** already automate some mail; list them before building
  agent rules that might fight existing filters.

## Counting and estimates

| Source | Trust |
|--------|--------|
| `messages.list` → `resultSizeEstimate` | **Low** — often sticks near ~201 even when the true set is huge |
| `+triage --format json` → `resultSizeEstimate` | Same underlying estimate |
| `labels.get` → `messagesTotal` / `messagesUnread` / `threadsTotal` | **Good** for a single label id (`INBOX`, `UNREAD`, `CATEGORY_SOCIAL`, user labels) |
| Paginated `messages.list` / `--page-all` counting ids | **Truth** for an arbitrary `q` |

`labels.list` does **not** include totals. Call `labels.get` per id.

## Scopes (practical)

| Scope family | Can do |
|--------------|--------|
| `gmail.readonly` | list/get/history/profile; no mutate |
| `gmail.modify` | readonly + labels/trash **and send/compose** |
| `gmail.compose` / `gmail.send` | send/draft paths |
| `gmail.insert` | `messages.import` / `insert` (not SMTP send) |
| `mail.google.com` | full mail (avoid unless necessary) |
| Settings / filters | often need modify or broader than pure readonly |
| `workflow +email-to-task` | also needs **Tasks** scope |

Testing-mode OAuth apps have a low scope cap on consumer accounts. Prefer a
short explicit `-s` list over “all recommended.”

## Dry-run

Most raw methods and helpers accept `--dry-run` (prints the HTTP request gws
would send). Use it to confirm URLs and bodies before mutate. Some workflow
helpers may still touch other APIs — treat dry-run as best-effort, not a
guarantee for every helper.
