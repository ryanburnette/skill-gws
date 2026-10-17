# Reading mail with gws

Start from [overview.md](./overview.md) for the full surface map.

## Prefer helpers for exploration

| Helper | Use |
|--------|-----|
| `gws gmail +triage` | from / subject / date / id — **no body** |
| `gws gmail +read --id ID` | decoded body; optional `--headers`, `--html` |
| `gws gmail +watch` | stream new mail (Pub/Sub; see [history-and-watch.md](./history-and-watch.md)) |

Write helpers need explicit user intent: [compose.md](./compose.md).

### +triage

Default query is **`is:unread`** (mailbox-wide), **not** Primary and not
inbox-only. Override with `--query`.

```bash
gws gmail +triage --max 20
gws gmail +triage --max 100 --query 'in:inbox category:promotions'
gws gmail +triage --max 5 --query 'is:unread in:inbox category:primary'
gws gmail +triage --max 20 --labels
gws gmail +triage --format json
```

Human output defaults to a **table** even though some help text mentions json as
the global default. Pass `--format json` for scripting.

JSON shape:

```json
{
  "query": "is:unread",
  "resultSizeEstimate": 201,
  "messages": [
    {
      "id": "…",
      "date": "…",
      "from": "…",
      "subject": "…",
      "labels": ["UNREAD", "INBOX"]
    }
  ]
}
```

`labels` appears when `--labels` is set.  
jq examples: `.messages[].subject`, `.messages[].id` — **not** `.[].subject`
(the root is an object).

`resultSizeEstimate` is as untrustworthy as on `messages.list` (see overview).

### +read

```bash
gws gmail +read --id MESSAGE_ID --headers
gws gmail +read --id MESSAGE_ID --format json
gws gmail +read --id MESSAGE_ID --html
```

JSON fields include: `thread_id`, `message_id` (RFC Message-ID, not Gmail id),
`from` `{name,email}`, `to[]`, `cc`, `subject`, `date`, `body_text`,
`body_html`, `references`.

Notes:

- Converts HTML-only mail to plain text when you ask for text.
- `--format text` may warn `unknown output format 'text'` and fall back; still
  prints the readable body. Prefer default invocation or `--format json`.
- Does not download file attachments ([attachments.md](./attachments.md)).

## Raw list / get

```bash
gws gmail users messages list --params '{
  "userId": "me",
  "maxResults": 50,
  "q": "in:inbox category:promotions"
}'

gws gmail users messages list --params '{
  "userId": "me",
  "labelIds": ["INBOX", "UNREAD"],
  "maxResults": 50
}'

gws gmail users messages get --params '{
  "userId": "me",
  "id": "MESSAGE_ID",
  "format": "metadata",
  "metadataHeaders": ["From", "To", "Subject", "Date", "Message-ID", "List-Id"]
}'

gws gmail users threads list --params '{"userId":"me","q":"…","maxResults":20}'
gws gmail users threads get --params '{
  "userId": "me",
  "id": "THREAD_ID",
  "format": "metadata",
  "metadataHeaders": ["From", "Subject", "Date"]
}'

gws gmail users getProfile --params '{"userId":"me"}'
```

### messages.list parameters

| Param | Notes |
|-------|--------|
| `q` | Gmail search syntax; not available with `gmail.metadata`-only scope |
| `labelIds` | **AND** — message must have every listed id |
| `includeSpamTrash` | default false |
| `maxResults` | default 100, API max **500** |
| `pageToken` | pagination |

List items are only `{ id, threadId }`. Everything else needs `get` or helpers.

### messages.get formats

| format | Payload | Typical use |
|--------|---------|-------------|
| `minimal` | ids, labels, snippet, size, historyId | cheapest non-list |
| `metadata` | + selected headers via `metadataHeaders` | triage-like |
| `full` | full MIME tree + body data / attachmentIds | read + attachments |
| `raw` | base64url RFC 2822 in `raw` | export / re-parse |

`internalDate` is epoch ms when Google accepted the message (more reliable than
the `Date` header for ordering).

### Pagination

```bash
gws gmail users messages list \
  --params '{"userId":"me","q":"in:inbox category:social","maxResults":500}' \
  --page-all --page-limit 50 --page-delay 100
```

`--page-all` emits **NDJSON** (one JSON object per page). Use for id collection,
not casual browsing.

## Threads

`threads.get` returns all messages in the thread. Labels can differ per message.
For UI-like “conversation,” threads help; for bulk ops on a search hit list,
prefer message ids from `messages.list`.

## Large mailbox habits

- Always set `maxResults` and a tight `q`
- `+triage` / `metadata` before `full` bodies
- `--page-all` only for intentional full-id collection
- Prefer `labels.get` for label totals; paginate for arbitrary queries
- Unbounded scans on 100k–1M+ mailboxes are slow and quota-heavy

## Scopes

- `gmail.readonly` — read only
- `gmail.modify` — read + labels/trash **and send**
- `gmail.metadata` — metadata without full body/`q` limitations per Google docs

See [overview.md](./overview.md) for counting pitfalls and safety defaults.
