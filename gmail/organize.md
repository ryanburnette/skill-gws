# Organize mail (labels, archive, read state)

Gmail has no separate “archive” or “mark read” RPC. Those are **label changes**
via `messages.modify`, `messages.batchModify`, or `threads.modify`.

## Common label operations

| Intent | Body fields |
|--------|-------------|
| Archive | `removeLabelIds: ["INBOX"]` |
| Move to inbox | `addLabelIds: ["INBOX"]` |
| Mark read | `removeLabelIds: ["UNREAD"]` |
| Mark unread | `addLabelIds: ["UNREAD"]` |
| Star | `addLabelIds: ["STARRED"]` |
| Unstar | `removeLabelIds: ["STARRED"]` |
| Trash (alt to `messages.trash`) | `addLabelIds: ["TRASH"]` (often with `INBOX` removed) |
| Apply user label | `addLabelIds: ["Label_…"]` |
| Remove user label | `removeLabelIds: ["Label_…"]` |

System label ids are stable strings: `INBOX`, `UNREAD`, `STARRED`, `IMPORTANT`,
`TRASH`, `SPAM`, `SENT`, `DRAFT`, `CATEGORY_PERSONAL` (Primary),
`CATEGORY_PROMOTIONS`, `CATEGORY_SOCIAL`, `CATEGORY_UPDATES`, `CATEGORY_FORUMS`,
plus star colors like `YELLOW_STAR`.

User labels use opaque ids (`Label_…`). Resolve name → id with `labels.list`.

## Single message

```bash
# Archive (dry-run first)
gws gmail users messages modify --dry-run \
  --params '{"userId":"me","id":"MESSAGE_ID"}' \
  --json '{"removeLabelIds":["INBOX"]}'

gws gmail users messages modify \
  --params '{"userId":"me","id":"MESSAGE_ID"}' \
  --json '{"removeLabelIds":["INBOX"],"addLabelIds":["Label_123"]}'
```

Limits: up to **100** label ids added and **100** removed per modify call.

## Batch (up to 1000 message ids)

```bash
gws gmail users messages batchModify \
  --params '{"userId":"me"}' \
  --json '{
    "ids": ["ID1","ID2"],
    "removeLabelIds": ["UNREAD"]
  }'
```

Successful `batchModify` responses are typically **empty** (no per-id report).
On failure, assume none or partial application is undefined — re-list to verify.

Prefer `batchModify` for bulk archive/mark-read/label. Prefer `messages.trash`
(or the trash runbook) when the goal is “remove to Trash,” which is clearer and
matches UI trash semantics.

## Threads vs messages

| Call | Effect |
|------|--------|
| `messages.modify` | Labels on **one** message |
| `threads.modify` | Same add/remove applied to **every** message in the thread |
| `threads.trash` | Entire thread → Trash |

Messages in one thread can carry **different** label sets. Thread-wide modify is
convenient and dangerous for mixed threads (e.g. only some messages still in
INBOX). For bulk cleanup of a search, trash/modify **message ids** from
`messages.list`, not thread ids, unless you have checked thread safety.

`threads.modify` body is only `addLabelIds` / `removeLabelIds` (no classification
label fields). `messages.modify` also accepts Workspace classification label
fields (usually irrelevant on consumer Gmail).

## Labels resource

```bash
# Names and ids (no counts)
gws gmail users labels list --params '{"userId":"me"}'

# Totals for one label
gws gmail users labels get --params '{"userId":"me","id":"INBOX"}'
gws gmail users labels get --params '{"userId":"me","id":"Label_123"}'

# Create user label
gws gmail users labels create \
  --params '{"userId":"me"}' \
  --json '{
    "name": "Receipts",
    "labelListVisibility": "labelShow",
    "messageListVisibility": "show"
  }'
```

Notes:

- Max **10,000** labels per mailbox (API).
- You cannot create/delete system labels.
- You generally cannot apply/remove `SENT` or `DRAFT` via modify the way you can
  `INBOX` / `UNREAD`.
- `labels.delete` removes the label and strips it from messages (messages stay).

## List by label id (AND)

`messages.list` `labelIds` is repeated and means **all** listed labels must match:

```bash
gws gmail users messages list --params '{
  "userId": "me",
  "labelIds": ["INBOX", "UNREAD"],
  "maxResults": 100
}'
```

Equivalent query form: `q: "in:inbox is:unread"`. User labels in `q`:
`label:Receipts` (display name), not the `Label_` id.

## Profile

```bash
gws gmail users getProfile --params '{"userId":"me"}'
# emailAddress, messagesTotal, threadsTotal, historyId
```

`messagesTotal` is mailbox-wide, not inbox-only.
