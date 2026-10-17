# Calendar surface overview

gws exposes Calendar two ways:

1. **Helpers** — `gws calendar +agenda`, `gws calendar +insert`
2. **Raw API** — `gws calendar <resource> <method> --params '…' [--json '…']`

Prefer `+agenda` for a human skim. Prefer raw methods when you need a window,
a single calendar, recurrence instances, free/busy, ACL, or exact fields.

Always invoke as `gws calendar …` (not bare `gws events` / `gws calendarList`).
Help text inside the Calendar tree often omits the `calendar` prefix.

Discover shapes anytime:

```bash
gws calendar --help
gws calendar events --help
gws schema calendar.events.list
gws schema calendar.Event
gws schema calendar.EventDateTime
```

Cross-service helpers that read Calendar: `gws workflow +weekly-digest`,
`+standup-report`, `+meeting-prep`.

## Topic map

| Topic | File |
|-------|------|
| Agenda, list/get, search, free/busy, settings | [reading.md](./reading.md) |
| Create, patch, Meet, recurrence, delete | [events.md](./events.md) |
| Calendar list vs calendars, ACL, colors | [calendars-and-sharing.md](./calendars-and-sharing.md) |
| Compact method table | [../reference/calendar-api-map.md](../reference/calendar-api-map.md) |

## Resource tree (raw)

```text
calendar
  +agenda          +insert
  calendarList     list get insert patch update delete watch
  calendars        get insert patch update delete clear transferOwnership
  events           list get insert patch update delete
                   instances import move quickAdd watch
  acl              list get insert patch update delete watch
  freebusy         query
  settings         list get watch
  colors           get
  channels         stop
```

`calendarId` is `primary` (the signed-in user's main calendar) or the real id
from `calendarList.list`. Secondary calendars usually look like
`HASH@group.calendar.google.com`. Holiday / subscribed calendars use their own
ids. gws URL-encodes them; pass the raw id in `--params`.

## Safety (Calendar-specific)

- Read-only (`+agenda`, `list` / `get`, `freebusy.query`, `settings`, `colors`)
  unless the user asks to mutate.
- **Never** `calendars.clear` — it deletes every event on the primary calendar.
- `calendars.delete` permanently removes a **secondary** calendar and its events.
  `calendarList.delete` only unscribes it from the sidebar.
- `+insert` and `events.insert` default to **not** emailing guests
  (`sendUpdates` defaults off). Set `sendUpdates` only when the user wants
  notifications. `--attendee` on `+insert` still adds guests if you run it
  for real.
- Do not add real attendees, ACL grants, or Meet links on a live calendar
  while exploring. Use a throwaway secondary calendar and delete it after.
- `outOfOffice` and `focusTime` can only be created on **primary**. Do not
  probe those types unless the user asked for an OOO / focus block.
- Empty DELETE responses: gws may write `download.html` in the cwd. Pass
  `--output ./empty.out` (path must stay **inside the current directory**).

## Scopes

| Scope family | Can do |
|--------------|--------|
| `calendar.readonly` | list/get/freebusy/settings/colors |
| `calendar.events` / `calendar.events.owned` | event CRUD on reachable calendars |
| `calendar.freebusy` / `calendar.events.freebusy` | `freebusy.query` |
| `calendar.calendarlist` | sidebar list only |
| `calendar.acls` | sharing rules |
| `calendar.calendars` | create/delete secondary calendars; `clear` |
| `https://www.googleapis.com/auth/calendar` | full Calendar (what `-s calendar` grants) |

Testing-mode OAuth apps have a low scope cap on consumer accounts. Prefer a
short explicit `-s` list. Full `calendar` is enough for this surface.

## Dry-run

Raw methods and `+insert` accept `--dry-run` (prints the HTTP request).
`+agenda` and the workflow Calendar helpers are read-only; `--dry-run` on
those still **runs** the reads and does not print the request.
