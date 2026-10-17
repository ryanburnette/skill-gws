# Compose, reply, forward, drafts

Mutating. Only run when the user explicitly wants outbound mail or a saved draft.

## Helpers (preferred)

| Helper | Purpose |
|--------|---------|
| `gws gmail +send` | New message |
| `gws gmail +reply` | Reply (sets threading headers, quotes original) |
| `gws gmail +reply-all` | Reply-all; `--remove` drops recipients |
| `gws gmail +forward` | Forward; original attachments on by default |

Shared options (see each `--help`):

- `--to` / `--cc` / `--bcc` (comma-separated where applicable)
- `--body` plain text, or HTML with `--html` (fragment tags; no full HTML doc)
- `-a` / `--attach` path (repeatable; combined size limit **25MB** on helpers)
- `--from` send-as alias (must already exist)
- `--draft` save draft instead of sending
- `--dry-run` show request

```bash
gws gmail +send --to alice@example.com --subject 'Hello' --body 'Hi'
gws gmail +send --to alice@example.com --subject 'Report' --body 'Attached' -a report.pdf
gws gmail +send --draft --to alice@example.com --subject 'WIP' --body '…'

gws gmail +reply --message-id MESSAGE_ID --body 'Thanks'
gws gmail +reply-all --message-id MESSAGE_ID --body 'Agreed' --remove bob@example.com
gws gmail +forward --message-id MESSAGE_ID --to dave@example.com --body 'FYI'
```

Helpers build RFC 5322 / MIME and base64 for you. Raw `messages.send` expects
you to supply a full encoded `raw` message (painful); use helpers unless you
have a reason not to.

### Draft via helper

`+send --draft` (and reply/forward `--draft`) posts to the **drafts** upload
endpoint rather than send. Inspect with `drafts.list` / `drafts.get`. Sending
later: `drafts.send` or the UI.

## Raw drafts

```bash
gws gmail users drafts list --params '{"userId":"me","maxResults":20}'
gws gmail users drafts get --params '{"userId":"me","id":"DRAFT_ID","format":"metadata"}'
gws gmail users drafts send --params '{"userId":"me"}' --json '{"id":"DRAFT_ID"}'
gws gmail users drafts delete --params '{"userId":"me","id":"DRAFT_ID"}'
# delete is permanent for that draft
```

Draft list entries look like `{ id, message: { id, threadId } }`.

## send vs import vs insert

| Method | Sends externally? | Notes |
|--------|-------------------|--------|
| `messages.send` / helpers | **Yes** | Real outbound mail |
| `messages.import` | No | Into this mailbox only; scanning/classification similar to SMTP receive; up to **150MB**; no SPF |
| `messages.insert` | No | Like IMAP APPEND; skips most scanning |

All three are distinct. “Put a copy in the mailbox” ≠ “send.”

## Send-as

```bash
gws gmail users settings sendAs list --params '{"userId":"me"}'
# helper: +send --from alias@example.com
```

Primary identity is always listed. Custom aliases need prior verification
(`sendAs.verify` when status is pending). Signature HTML can be set via
sendAs patch/update (Gmail sanitizes HTML).

## Scopes

Send paths accept `gmail.send`, `gmail.compose`, or `gmail.modify` (among
others). **`gmail.modify` is enough to send** — important for least-privilege
agent design: a token that can archive can often also send.
