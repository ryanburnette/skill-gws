# Destructive Gmail ops — policy

| Action | Recoverable? | Default stance |
|--------|--------------|----------------|
| `messages.trash` / `threads.trash` | Yes (~30d Trash) | **Preferred** for bulk remove |
| `modify` add `TRASH` / remove `INBOX` | Yes (archive or trash-like) | OK when intentional |
| remove label / archive (`modify`) | Yes | OK when intentional |
| `messages.delete` / `batchDelete` | **No** | Only on explicit user request |
| `threads.delete` | **No** (whole thread) | Only on explicit user request |
| `drafts.delete` | No for that draft | Confirm |
| `filters.delete` | No for that rule | Confirm |
| `+send` / reply / forward | N/A (external side effect) | Only on explicit request |

## Bulk remove checklist

1. Agree query (tab vs category-wide).
2. Paginated real count (do not trust `resultSizeEstimate`).
3. Metadata sample for sanity (`+triage` or `messages.get` metadata).
4. Trash **messages** (not threads) unless threads proven safe.
5. Verify query empty; spot-check other tabs.
6. Document the outcome in the **host/project** log or runbook (not necessarily this skill).

See [runbooks/trash-by-gmail-query.md](../runbooks/trash-by-gmail-query.md).

## Related

- Organize without deleting: [organize.md](./organize.md)
- History will show trash as label changes, not `messagesDeleted`: [history-and-watch.md](./history-and-watch.md)
