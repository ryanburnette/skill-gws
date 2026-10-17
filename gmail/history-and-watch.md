# History sync and watch

Avoid re-listing the whole mailbox. Use **history** for incremental changes and
optional **Pub/Sub watch** for push.

## history.list

```bash
# startHistoryId: from getProfile, a prior history response, or a message/thread
gws gmail users history list --params '{
  "userId": "me",
  "startHistoryId": "HISTORY_ID",
  "maxResults": 100
}'
```

Optional filters:

| Param | Values / use |
|-------|----------------|
| `historyTypes` | `messageAdded`, `messageDeleted`, `labelAdded`, `labelRemoved` (repeatable) |
| `labelId` | Only records involving this label |
| `maxResults` | Default 100, max **500** |
| `pageToken` | Pagination |

### Record shape

Each history entry may include:

- `messages` — sparse `{id, threadId}` (prefer typed fields below)
- `messagesAdded` — new mail (includes `labelIds` on the message snapshot)
- `messagesDeleted` — **permanent** delete, not trash
- `labelsAdded` / `labelsRemoved` — `{ labelIds, message: { id, threadId, labelIds? } }`

Trash often appears as **label** changes (`TRASH` added, `INBOX` removed), not
`messagesDeleted`.

### Lifecycle rules

- History ids increase over time but are **not contiguous**.
- A `startHistoryId` that is too old or invalid → **HTTP 404**. Full re-sync
  (list current state, store a fresh `historyId`).
- Docs: ids are typically valid **at least a week**, sometimes only hours under
  rare conditions.
- After a successful page, store the response `historyId` for the next poll.
- No `nextPageToken` and empty `history` ⇒ nothing new; still keep the returned
  `historyId`.

```bash
gws gmail users getProfile --params '{"userId":"me"}'
# read .historyId as the high-water mark after a full sync
```

## users.watch + users.stop (push)

API-level push publishes to **Cloud Pub/Sub**. The topic must:

1. Already exist in the **same GCP project** that owns the OAuth client / watch
2. Grant Gmail publish permission on the topic

```bash
gws gmail users watch --params '{"userId":"me"}' --json '{
  "topicName": "projects/PROJECT_ID/topics/TOPIC",
  "labelIds": ["INBOX"],
  "labelFilterBehavior": "include"
}'

gws gmail users stop --params '{"userId":"me"}'
```

Watch **expires (~7 days)**; renew before expiry. Push notifications are thin
(mailbox change signals) — you still call `history.list` to learn what changed.

## Helper: `gws gmail +watch`

Higher-level streamer: can create or reuse Pub/Sub resources, pull, and emit
Gmail messages as NDJSON.

```bash
gws gmail +watch --help
gws gmail +watch --project PROJECT_ID --label-ids INBOX --once
gws gmail +watch --project PROJECT_ID --cleanup --output-dir ./emails
```

Useful flags: `--subscription`, `--topic`, `--poll-interval`, `--msg-format`,
`--max-messages`, `--once`, `--cleanup`, `--output-dir`.

Requires working GCP credentials/project permissions for Pub/Sub, not only
Gmail OAuth.

## Agent-oriented pattern

1. Full sync once (or bounded query sync); store `historyId` from profile or
   last history response.
2. On timer or Pub/Sub nudge: `history.list` from stored id.
3. For `messagesAdded`, optionally `messages.get` format `metadata` or `+read`.
4. On 404: full sync again.
5. Never rely on `resultSizeEstimate` for “how much is new.”
