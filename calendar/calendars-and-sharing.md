# Calendars, the sidebar, and ACL

Two resources people conflate:

| Resource | What it is |
|----------|------------|
| `calendars` | The calendar object: summary, description, time zone, conference defaults |
| `calendarList` | The signed-in user's **sidebar**: color, hidden, selected, default reminders, `accessRole` |

`calendars.insert` creates a secondary calendar (caller becomes data owner)
and usually adds it to the list. `calendarList.insert` only subscribes to an
**existing** id.

## List the sidebar

```bash
gws calendar calendarList list --params '{"maxResults":50,"showHidden":true}'
gws calendar calendarList get --params '{"calendarId":"primary"}'
gws calendar calendars get --params '{"calendarId":"primary"}'
```

Default `calendarList.list` **omits hidden** calendars. Pass `showHidden: true`.
`showDeleted: true` includes unsubscribed rows (`deleted: true`).

Page size default 100, max 250. `nextSyncToken` on the last page.

Useful list fields: `id`, `summary`, `summaryOverride`, `primary`,
`accessRole`, `selected`, `hidden`, `colorId`, `backgroundColor`,
`timeZone`, `defaultReminders`, `conferenceProperties`.

`hidden: true` is omitted when the calendar is visible. `selected: false`
is omitted when unselected. Do not treat a missing flag as unknown — treat
missing `hidden` as visible.

`minAccessRole` filters the list: `freeBusyReader` · `reader` ·
`writerWithoutPrivateAccess` · `writer` · `owner`.

`+agenda` uses this list. Hidden calendars are easy to miss unless you pass
`--calendar`.

## accessRole vs data owner

`accessRole` is what **this user** can do on that calendar:

| Role | Meaning |
|------|---------|
| `freeBusyReader` | busy blocks only |
| `reader` | see events; private details hidden |
| `writerWithoutPrivateAccess` | write, but private details hidden |
| `writer` | read/write; private details visible |
| `owner` | writer + manage ACL |

`owner` is not the same as **data owner**. A calendar has one data owner
and may have several `owner`-role users. `calendars.transferOwnership` is
Workspace-admin only (`useAdminAccess: true`) and does not apply to
consumer accounts.

Primary `calendars.get` reports `conferenceProperties.allowedConferenceSolutionTypes`
(often `hangoutsMeet`). Its `id` is usually the account email, but `primary`
works as an alias everywhere a `calendarId` is required.

## Create / tweak / delete a calendar

```bash
# secondary calendar
gws calendar calendars insert --json '{
  "summary":"Scratch",
  "description":"throwaway",
  "timeZone":"America/New_York"
}'

# calendar metadata (title, description, tz)
gws calendar calendars patch \
  --params '{"calendarId":"CAL_ID"}' \
  --json '{"description":"updated"}'

# sidebar only (color, hidden, selected, summaryOverride)
gws calendar calendarList patch \
  --params '{"calendarId":"CAL_ID"}' \
  --json '{"hidden":true,"selected":false,"colorId":"5"}'

# RGB colors need colorRgbFormat=true
gws calendar calendarList patch \
  --params '{"calendarId":"CAL_ID","colorRgbFormat":true}' \
  --json '{"backgroundColor":"#0088aa","foregroundColor":"#ffffff"}'
```

A new secondary calendar typically lands `accessRole: owner`, `selected:
true`, `hidden: false`, empty `defaultReminders`.

```bash
# unsubscribe (calendar remains)
gws calendar calendarList delete --params '{"calendarId":"CAL_ID"}' --output ./empty.out

# destroy a secondary calendar and its events
gws calendar calendars delete --params '{"calendarId":"CAL_ID"}' --output ./empty.out
```

**Never** `calendars.clear` — it wipes the primary calendar.

`calendars.delete` on a missing id is 404. Same empty-body / `--output`
rule as [events.md](./events.md).

## Sharing (ACL)

```bash
gws calendar acl list --params '{"calendarId":"CAL_ID"}'
gws calendar acl insert \
  --params '{"calendarId":"CAL_ID","sendNotifications":false}' \
  --json '{
    "role":"reader",
    "scope":{"type":"user","value":"user@example.com"}
  }'
```

`sendNotifications` defaults **true**. Turn it off unless the user wants a
share email.

| `role` | Access |
|--------|--------|
| `none` | revoke |
| `freeBusyReader` | busy only |
| `reader` | read; private details hidden |
| `writerWithoutPrivateAccess` | write; private details hidden |
| `writer` | write + see private + read ACL |
| `owner` | manage ACL |

`scope.type`: `default` (public), `user`, `group`, `domain`. `user` /
`group` need `scope.value` (email). Rule ids look like `user:email`.

Do not insert `type=default` (public) or real people while exploring.

Primary often has a single `owner` + `user` rule for the account. A fresh
secondary may show two `owner` user rules; list before you patch.

## Watch / channels

`events.watch`, `calendarList.watch`, `acl.watch`, `settings.watch` need a
webhook `Channel` (`type: web_hook`, `address`, `id`). `channels.stop`
takes that channel body. Do not open a watch unless the user asked and a
receiver exists.

## Exploring safely

1. `calendarList.list` with `showHidden: true`.
2. If you must mutate, `calendars.insert` a throwaway, work there, then
   `calendars.delete` it.
3. Leave primary, subscribed, and shared calendars alone.
