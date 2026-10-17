# Settings and filters

Read account automation and identity before building new agent rules. Filters
already rewrite labels and may forward or trash mail.

## Filters

```bash
gws gmail users settings filters list --params '{"userId":"me"}'
gws gmail users settings filters get --params '{"userId":"me","id":"FILTER_ID"}'
```

Cap: **1,000** filters per user (API).

### Criteria (`FilterCriteria`)

| Field | Meaning |
|-------|---------|
| `from` | Sender display name or address |
| `to` | Matches to/cc/bcc; local-part forms allowed |
| `subject` | Case-insensitive phrase |
| `query` | Full Gmail search syntax |
| `negatedQuery` | Must not match |
| `hasAttachment` | boolean |
| `excludeChats` | boolean |
| `size` + `sizeComparison` | Size predicate |

### Actions (`FilterAction`)

Only three action fields exist in the API:

| Field | Use |
|-------|-----|
| `addLabelIds` | Apply labels — including `TRASH`, `STARRED`, user labels |
| `removeLabelIds` | Strip labels — **archive** = remove `INBOX`; **mark read** = remove `UNREAD` |
| `forward` | Forward to an address (must be allowed / verified forwarding path) |

There is no separate `markAsRead` or `trash` action key. Encode those as label
add/remove.

```bash
# Example create (mutate — only with intent)
gws gmail users settings filters create \
  --params '{"userId":"me"}' \
  --json '{
    "criteria": { "from": "news@example.com" },
    "action": { "removeLabelIds": ["INBOX"] }
  }'
```

`filters.delete` is immediate and permanent for that rule.

Filters apply to **messages** as they arrive, not retroactively to the whole
thread history (API description).

## Vacation, IMAP, POP, language, auto-forward

```bash
gws gmail users settings getVacation --params '{"userId":"me"}'
gws gmail users settings getImap --params '{"userId":"me"}'
gws gmail users settings getPop --params '{"userId":"me"}'
gws gmail users settings getLanguage --params '{"userId":"me"}'
gws gmail users settings getAutoForwarding --params '{"userId":"me"}'
```

Matching `update*` methods exist. Treat updates as high-impact account changes.

## Forwarding addresses

```bash
gws gmail users settings forwardingAddresses list --params '{"userId":"me"}'
```

Lists verified (or pending) forwarding targets. Auto-forward enablement is
separate (`getAutoForwarding` / `updateAutoForwarding`) and requires a verified
address.

## Send-as

See [compose.md](./compose.md). `sendAs.list` / `get` / `patch` / `verify` /
`smimeInfo`.

## Delegates and CSE

- **delegates**: list/create/delete often restricted to **service accounts with
  domain-wide delegation** — expect 403 on consumer Gmail user OAuth.
- **cse** (client-side encryption): identities/keypairs; specialized Workspace
  setups.

## Scopes

Settings and filters generally need more than `gmail.readonly`. If list calls
403, re-auth with modify (or the specific settings scopes your app requests).
