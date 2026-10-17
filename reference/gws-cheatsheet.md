# gws cheatsheet

```bash
export GOOGLE_WORKSPACE_CLI_KEYRING_BACKEND=file   # headless Linux

gws --version
gws auth status
gws auth login -s gmail,drive,calendar,sheets,docs

# Profile / triage / read
gws gmail users getProfile --params '{"userId":"me"}'
gws gmail +triage --max 10 --query 'in:inbox category:primary'
gws gmail +triage --max 20 --labels --format json
gws gmail +read --id ID --headers

# Counts (labels.get — not resultSizeEstimate)
gws gmail users labels get --params '{"userId":"me","id":"INBOX"}'

# Organize (mutate)
gws gmail users messages modify \
  --params '{"userId":"me","id":"ID"}' \
  --json '{"removeLabelIds":["INBOX"]}'

# History
gws gmail users history list --params '{"userId":"me","startHistoryId":"HID","maxResults":100}'

# Filters (read)
gws gmail users settings filters list --params '{"userId":"me"}'

# Schema
gws schema gmail.users.messages.list
gws schema gmail.users.messages.modify
gws schema gmail.FilterAction

gws drive files list --params '{"pageSize":10}'
gws drive about get --params '{"fields":"canCreateDrives,user(emailAddress)"}'
gws drive permissions list --params '{"fileId":"FILE_ID","fields":"permissions(id,type,role,emailAddress,permissionDetails)"}'

# Calendar
gws calendar +agenda --today --format json
gws calendar calendarList list --params '{"maxResults":50,"showHidden":true}'
gws calendar events list --params '{
  "calendarId":"primary",
  "singleEvents":true,
  "orderBy":"startTime",
  "timeMin":"2026-08-20T00:00:00-04:00",
  "timeMax":"2026-08-27T00:00:00-04:00"
}'
gws calendar freebusy query --json '{
  "timeMin":"2026-08-20T00:00:00-04:00",
  "timeMax":"2026-08-21T00:00:00-04:00",
  "items":[{"id":"primary"}]
}'
```

Drive sharing: [../drive/sharing.md](../drive/sharing.md). Shared drives:
[../drive/my-drive-vs-shared-drives.md](../drive/my-drive-vs-shared-drives.md).
Calendar: [../calendar/overview.md](../calendar/overview.md).

## Gmail q fragments

`is:unread` `newer_than:7d` `older_than:1y` `from:` `to:` `subject:`  
`has:attachment` `filename:pdf` `in:inbox` `in:trash` `in:anywhere`  
`category:primary|promotions|social|updates|forums`  
`label:NAME` `-category:promotions`

More: [gmail-queries.md](./gmail-queries.md). Gmail deep dive: [../gmail/overview.md](../gmail/overview.md).

## Bulk id collect

```bash
gws gmail users messages list \
  --params '{"userId":"me","q":"…","maxResults":500}' \
  --page-all --page-limit N --page-delay MS
```

`--page-limit N` stops after N pages **silently**. A result whose count equals
`maxResults × N` exactly is probably truncated, not complete. Check the time
span the records cover before trusting a total.
