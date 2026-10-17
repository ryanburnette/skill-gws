# Reading calendars and events

Start from [overview.md](./overview.md). Writes: [events.md](./events.md).

## Prefer +agenda for a skim

```bash
gws calendar +agenda
gws calendar +agenda --today
gws calendar +agenda --tomorrow
gws calendar +agenda --week --format table
gws calendar +agenda --days 3 --calendar 'Team'
gws calendar +agenda --today --timezone America/New_York
gws calendar +agenda --format json
```

Read-only. Walks **every calendar on the list** unless `--calendar` is set.
`--calendar` accepts a visible name or an id. Matching is loose — a short
name can hit more than one calendar if several names share that string.
Use the exact calendar id when you need one calendar.

Timezone comes from Calendar **settings** (`timezone`), cached about a day
(upstream). `--timezone` / `--tz` overrides the window, not the event
timestamps already stored on the event.

### Windows

| Flag | Window |
|------|--------|
| *(default)* or `--days 1` | Next 24 hours from now |
| `--days N` | Next N days from now |
| `--today` | Account-timezone calendar day |
| `--tomorrow` | Next calendar day |
| `--week` | Next 7 days from now — **not** Monday–Sunday |

`--today` / `--tomorrow` use midnight–midnight in the chosen zone. Default
and `--week` are rolling from the current instant.

### JSON shape

```json
{
  "count": 2,
  "timeMin": "2026-08-20T00:00:00-04:00",
  "timeMax": "2026-08-21T00:00:00-04:00",
  "events": [
    {
      "calendar": "Team",
      "summary": "Standup",
      "start": "2026-08-20T09:00:00-04:00",
      "end": "2026-08-20T09:15:00-04:00",
      "location": ""
    }
  ]
}
```

All-day rows use a date for `start` / `end` (end is exclusive). Recurring
series are expanded into instances. Fields are the skim only — no attendees,
description, hangout, or event id. Use `events.get` / `events.list` for those.

`--format table` is nicer for humans. Pass `--format json` for scripting.

## Raw events.list / get

```bash
gws calendar events list --params '{
  "calendarId":"primary",
  "singleEvents":true,
  "orderBy":"startTime",
  "timeMin":"2026-08-20T00:00:00-04:00",
  "timeMax":"2026-08-27T00:00:00-04:00",
  "maxResults":50
}'

gws calendar events get --params '{
  "calendarId":"primary",
  "eventId":"EVENT_ID"
}'
```

`calendarId` is required. Missing it is a local validation error.

### Time filters

`timeMin` / `timeMax` must be RFC3339 **with a zone offset** (`Z` or
`-04:00`). A naive `2026-08-20T00:00:00` returns API `badRequest`.

- `timeMin` — exclusive lower bound on the event's **end**
- `timeMax` — exclusive upper bound on the event's **start**

Default response zone is the calendar's `timeZone`. Override with `timeZone`.

### Recurrence

`singleEvents` defaults **false**: you get the master series row
(`recurrence: ["RRULE:…"]`), not the occurrences.

Set `singleEvents: true` to expand instances. Each instance has
`recurringEventId` and an id like `MASTERID_20261214T140000Z`.

`orderBy: "startTime"` **requires** `singleEvents: true`. Otherwise:

> The requested ordering is not available for the particular query.

`orderBy: "updated"` does not need expansion. Default order is unspecified
but stable.

`events.instances` lists occurrences of one series (`eventId` = master id).
Optional `timeMin` / `timeMax` / `maxResults` (default 250, max 2500).

### Search and type filters

`q` is free-text over summary, description, location, attendee name/email,
organizer name/email.

```bash
gws calendar events list --params '{
  "calendarId":"primary",
  "q":"standup",
  "singleEvents":true,
  "orderBy":"startTime",
  "timeMin":"2026-08-01T00:00:00Z",
  "timeMax":"2026-09-01T00:00:00Z"
}'
```

`eventTypes` is repeatable. A JSON string or array both work:

`"default"` · `"birthday"` · `"focusTime"` · `"fromGmail"` ·
`"outOfOffice"` · `"workingLocation"`

Omitting `eventTypes` returns only `default`. Out of office, focus time,
working location, birthdays, and from-Gmail events are absent from the list
until you pass those types.

`iCalUID` finds an event by its iCalendar UID (what `events.import` needs).
That is not the same as the Google `id`.

`showDeleted: true` includes `status: "cancelled"` rows. `events.get` on a
deleted event also returns the cancelled resource, not 404.

`fields` is accepted inside `--params` even though discovery may not list it:

```bash
gws calendar events list --params '{
  "calendarId":"primary",
  "maxResults":10,
  "fields":"items(id,summary,start,end,status,eventType),nextPageToken"
}'
```

### Pagination

Default page size is 250; max 2500. `--page-all` emits NDJSON (one page per
line). Bound `--page-limit`. The last page can be empty with no
`nextPageToken`.

`nextSyncToken` (full sync, last page only) is the incremental hook. A stale
token returns **410**; drop it and resync. `syncToken` cannot be mixed with
most filters (`q`, `timeMin`, `eventTypes`, …).

## Free/busy

```bash
gws calendar freebusy query --json '{
  "timeMin":"2026-08-20T00:00:00-04:00",
  "timeMax":"2026-08-21T00:00:00-04:00",
  "timeZone":"America/New_York",
  "items":[{"id":"primary"},{"id":"HASH@group.calendar.google.com"}]
}'
```

POST body, not `--params`. `timeMin` is required. Response
`calendars[ID].busy` is `[{start,end}, …]` in the requested zone. Opaque
blocks only — no titles. Missing `timeMin` → `Missing timeMin parameter.`

## Settings and colors

```bash
gws calendar settings list
gws calendar settings get --params '{"setting":"timezone"}'
gws calendar colors get
```

Useful setting ids: `timezone`, `locale`, `weekStart` (`0` = Sunday),
`defaultEventLength` (minutes), `format24HourTime`, `showDeclinedEvents`,
`hideInvitations`, `autoAddHangouts`, `dateFieldOrder`.

`colors.get` returns `calendar` and `event` palettes keyed by `colorId`
(`background` / `foreground` hex). `calendarList` entries and events store
the id, not the hex.

## Workflow helpers (read-only)

```bash
gws workflow +weekly-digest --format json
gws workflow +standup-report --format json    # also needs Tasks scope
gws workflow +meeting-prep                    # next event on --calendar (default primary)
```

`+weekly-digest` uses a rolling week from now plus a Gmail unread estimate
(same untrustworthy `resultSizeEstimate` as Gmail). `+meeting-prep` JSON
includes `summary`, `start`, `end`, `location`, `description`, `attendees`,
`attendeeCount`, `htmlLink`, `hangoutLink`. `--dry-run` still executes.
