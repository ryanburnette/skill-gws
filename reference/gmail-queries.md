# Gmail query recipes

`q` on `messages.list` / `threads.list` / `+triage --query` uses the same
language as the Gmail search box.

## Tabs (inbox + category)

```text
in:inbox category:primary
in:inbox category:promotions
in:inbox category:social
in:inbox category:updates
in:inbox category:forums
```

## Unread / location

```text
is:unread
is:unread in:inbox
is:unread in:inbox category:primary
in:inbox
in:sent
in:drafts
in:trash
in:spam
in:anywhere
is:starred
is:important
is:chat
```

`category:social` without `in:inbox` includes **archived** social mail.  
`in:anywhere` searches across folders; combine carefully.

## Time and size

```text
newer_than:1d
newer_than:7d
older_than:365d
larger:5M
smaller:100K
```

## People and content

```text
from:alice@example.com
to:bob@example.com
from:github.com
subject:invoice
subject:(receipt OR invoice)
"exact phrase"
filename:pdf
has:attachment
has:userlabels
list:noreply.github.com
rfc822msgid:<…@example.com>
deliveredto:you@example.com
label:Receipts
```

## Boolean and exclusion

```text
from:a@example.com OR from:b@example.com
from:github.com (SSH OR security)
in:inbox -category:promotions -category:social
is:unread -category:promotions
newer_than:30d has:attachment filename:pdf
```

## Ops / cleanup patterns

```text
# Tab to clear (example)
in:inbox category:social

# Recent promotions anywhere
category:promotions newer_than:7d

# Inspect what landed in trash
in:trash newer_than:1d
in:trash category:social

# Everything in a category except trash/spam
category:social -in:trash -in:spam
```

## Pitfalls

- `+triage` default is `is:unread`, not Primary.
- `resultSizeEstimate` is often wrong; paginate or use `labels.get`.
- `label:Name` uses the **display name**; modify APIs need the `Label_…` id.
- `labelIds` on `messages.list` is AND of system/user **ids**, not `q` syntax.
- Quotes and parentheses matter; pass `q` inside proper JSON string escaping.

## Operators worth knowing

Official operator list (Google): search help for “Gmail search operators.”
Commonly useful beyond the above: `cc:`, `bcc:`, `before:yyyy/mm/dd`,
`after:yyyy/mm/dd`, `has:drive`, `has:document`, `filename:`, `newer_than:`,
`older_than:`.
