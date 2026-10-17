# Gmail categories vs UI tabs

UI tabs are **search filters** on Inbox + category labels, not separate stores.

| UI tab | Search closest to the tab |
|--------|---------------------------|
| Primary | `in:inbox category:primary` |
| Promotions | `in:inbox category:promotions` |
| Social | `in:inbox category:social` |
| Updates | `in:inbox category:updates` |
| Forums | `in:inbox category:forums` |

Underlying **tab** system labels: `CATEGORY_PERSONAL` (Primary),
`CATEGORY_PROMOTIONS`, `CATEGORY_SOCIAL`, `CATEGORY_UPDATES`, `CATEGORY_FORUMS`.
Those appear on `labels.list` and on `messages.get` → `labelIds`.

## Extra categories (real, not tabs)

`category:purchases` and `category:reservations` are **real Gmail categories**
(the same `category:` search family). They are not inbox tabs and they are
**not** exposed as `CATEGORY_PURCHASES` / `CATEGORY_RESERVATIONS` on
`labels.list` or `labelIds`. Detect them with search, not by reading labels.

| Search | Typical mail | UI |
|--------|----------------|-----|
| `category:purchases` | Receipts, order confirmations, some shipping | Left-nav Purchases + package cards (not a top tab) |
| `category:reservations` | Travel / hotel bookings | Same idea |

A message can be **in two categories at once**: e.g. Purchases *and* Updates
(`CATEGORY_UPDATES` on the message, and it matches `category:purchases`).
Observed: almost all recent purchases hits also had Updates; one Home Depot
order was Purchases + Primary. Newsletters in Updates (Rumble, etc.) do not
match Purchases.

Inbox Purchases = `in:inbox category:purchases`. Archive (drop `INBOX`) and it
leaves that inbox view; the purchases category on the message remains.

The five tab categories are mutually exclusive in practice (one
`CATEGORY_*` label). Purchases/reservations stack *on top* of a tab category.
You cannot add/remove Purchases via `addLabelIds` the way you can
`CATEGORY_UPDATES`. Fuzzy: USPS Informed Delivery and some statements match.

Unrelated: Workspace **classification labels** on `messages.modify` are Drive
Labels for orgs, not these inbox categories.

## Critical distinctions

| Query | Meaning |
|-------|---------|
| `in:inbox category:social` | **Social tab** (what users usually mean) |
| `category:social` | All Social-categorized mail, **including archived** |
| `is:unread` | All unread matching other terms — not tab-scoped by default |

`gws gmail +triage` default query is unread-oriented (`is:unread` unless
`--query` overrides). It does **not** default to Primary tab.

## Counting

`messages.list` → `resultSizeEstimate` is **often wrong** (observed stuck near
~200 while a Social tab had ~750 messages). Always paginate for truth:

```bash
gws gmail users messages list \
  --params '{"userId":"me","q":"in:inbox category:social","maxResults":500}' \
  --page-all --page-limit 50 --page-delay 100
```

Thread count and message count can differ on the same query.

## Default interpretation

When a user refers to a tab name, use `in:inbox category:<tab>` unless they
explicitly want category-wide including archive.

## Counts for a category label

`labels.get` on `CATEGORY_SOCIAL` (etc.) counts **category-wide**, including
archived mail — not the tab. Tab-sized counts still need a paginated
`messages.list` on `in:inbox category:…`.

Related: [organize.md](./organize.md) (label ids), [reading.md](./reading.md).
