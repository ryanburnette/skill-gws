# Runbook: trash all messages matching a Gmail query

Recoverable (~30 days). Do not permanent-delete unless explicitly ordered.

Requires `gws` on `PATH` and a working login (file keyring on headless Linux:
`GOOGLE_WORKSPACE_CLI_KEYRING_BACKEND=file`).

## 1. Query

Tab example:

```text
in:inbox category:social
```

## 2. Real count (page; ignore resultSizeEstimate)

```bash
export GOOGLE_WORKSPACE_CLI_KEYRING_BACKEND=file

gws gmail users messages list \
  --params '{"userId":"me","q":"QUERY_HERE","maxResults":500}' \
  --page-all --page-limit 50 --page-delay 100 \
  > /tmp/msg-pages.jsonl
```

Collect unique ids with any JSON tool you trust (Node example):

```bash
node -e '
const fs = require("fs");
const ids = [];
const threads = new Set();
for (const line of fs.readFileSync("/tmp/msg-pages.jsonl","utf8").split("\n")) {
  if (!line.trim()) continue;
  const o = JSON.parse(line);
  for (const m of o.messages || []) {
    ids.push(m.id);
    if (m.threadId) threads.add(m.threadId);
  }
}
const uniq = [...new Set(ids)];
console.log("messages", ids.length, "unique", uniq.length, "threads", threads.size);
fs.writeFileSync("/tmp/msg-ids.txt", uniq.join("\n") + (uniq.length ? "\n" : ""));
'
```

## 3. Sample headers

```bash
gws gmail +triage --max 15 --query 'QUERY_HERE' --format json
```

## 4. Trash via API (fast path)

Refresh access token from `gws auth export --unmasked` **in memory only**, then:

`POST https://gmail.googleapis.com/gmail/v1/users/me/messages/{id}/trash`

Parallelism ~10–12; retry once on HTTP 429. Log only ok/fail counts, never tokens.

Equivalent slow path: loop
`gws gmail users messages trash --params '{"userId":"me","id":"…"}'`.

**Do not** call delete/batchDelete in this runbook.

## 5. Verify

```bash
gws gmail users messages list --params \
  '{"userId":"me","q":"QUERY_HERE","maxResults":5}'
# empty messages / estimate 0

gws gmail +triage --max 5 --query 'in:trash …'   # optional
```

## Rollback

Gmail UI → Trash → Move to Inbox (or `messages.untrash`).
