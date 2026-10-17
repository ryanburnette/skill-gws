# Helpers vs raw API

Full topic index: [overview.md](./overview.md). Compact method table:
[../reference/gmail-api-map.md](../reference/gmail-api-map.md).

## Invocation shapes

```text
gws gmail +triage …
gws gmail +read --id …
gws gmail users messages list --params '{…}'
gws gmail users messages modify --params '{…}' --json '{…}'
gws schema gmail.users.messages.modify
```

Help text sometimes shows `gws users …` inside the gmail command tree. Prefer
explicit `gws gmail users …`.

## When to use which

| Goal | Prefer |
|------|--------|
| Skim unread / search hits | `+triage` |
| Read one body | `+read` |
| Send / reply / forward / draft | `+send` / `+reply` / `+forward` |
| Archive, mark read, label | raw `modify` / `batchModify` |
| True label counts | `labels.get` |
| Incremental changes | `history.list` |
| Attachment bytes | `messages.attachments.get` |
| Filters / vacation / send-as | `users.settings.*` |
| Bulk trash by query | [../runbooks/trash-by-gmail-query.md](../runbooks/trash-by-gmail-query.md) |

## Risk ladder

| Method class | Risk |
|--------------|------|
| list / get / +triage / +read / history / labels.list\|get | read |
| modify / batchModify / labels.create | mutate (reversible-ish) |
| trash / untrash | recoverable mutate (~30d) |
| +send / reply / forward / drafts.send / messages.send | **sends mail** |
| filters.create\|delete / settings.update* | account policy |
| drafts.delete / labels.delete | permanent for that object |
| messages.delete / batchDelete / threads.delete | **permanent mail loss** |

## Output

- Raw methods: JSON by default; `--format table|yaml|csv` where supported
- `+triage`: table by default; `--format json` → `{ query, messages, resultSizeEstimate }`
- `+read`: human text by default; `--format json` for structured fields
- `--page-all`: NDJSON (one page per line)
- `--dry-run`: print request, do not execute (best-effort on helpers)
