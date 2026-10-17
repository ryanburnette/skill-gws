# Create, update, and delete events

Start from [overview.md](./overview.md). Reads: [reading.md](./reading.md).

Mutate only with explicit user intent. Prefer a throwaway secondary calendar
when exploring.

## +insert (helper)

```bash
gws calendar +insert \
  --summary 'Standup' \
  --start '2026-08-21T09:00:00-04:00' \
  --end '2026-08-21T09:15:00-04:00'

gws calendar +insert --calendar CALENDAR_ID \
  --summary 'Review' \
  --start '2026-08-21T10:00:00-04:00' \
  --end '2026-08-21T10:30:00-04:00' \
  --location 'Room A' \
  --description 'Agenda…' \
  --attendee user@example.com \
  --meet
```

`--summary`, `--start`, `--end` are required. Times are RFC3339. Default
calendar is `primary`. `--attendee` is repeatable.

`--meet` sends `conferenceDataVersion=1` and a `hangoutsMeet` `createRequest`
(random `requestId`). The response includes `hangoutLink` and
`conferenceData.entryPoints` when the create succeeds.

`--dry-run` prints the POST. `sendUpdates` is **not** set, so the API
default is no guest mail. Some mail can still go out; do not add real
attendees unless the user asked.

## Raw insert

```bash
gws calendar events insert --params '{"calendarId":"primary"}' --json '{
  "summary":"Standup",
  "description":"optional HTML ok",
  "location":"Room A",
  "start":{"dateTime":"2026-08-21T09:00:00","timeZone":"America/New_York"},
  "end":{"dateTime":"2026-08-21T09:15:00","timeZone":"America/New_York"}
}'
```

`start` and `end` are required. Both must be `date` (all-day) or both
`dateTime`. Mixing them → `Start and end times must either both be date or
both be dateTime.` Missing end → `Missing end time.`

Timed events: `dateTime` with an offset **or** a local time plus `timeZone`
(IANA). The API normalizes to offset + zone. Recurring events should set
`timeZone` on start/end.

All-day: `{"date":"2026-08-24"}` … `{"date":"2026-08-25"}`. End is
**exclusive**.

Reminders default to the calendar's `defaultReminders` (`useDefault: true`).

### Meet without the helper

```bash
gws calendar events insert \
  --params '{"calendarId":"primary","conferenceDataVersion":1}' \
  --json '{
    "summary":"Sync",
    "start":{"dateTime":"2026-08-21T11:00:00-04:00","timeZone":"America/New_York"},
    "end":{"dateTime":"2026-08-21T11:30:00-04:00","timeZone":"America/New_York"},
    "conferenceData":{
      "createRequest":{
        "requestId":"unique-per-attempt",
        "conferenceSolutionKey":{"type":"hangoutsMeet"}
      }
    }
  }'
```

`conferenceDataVersion` defaults to **0**, which **ignores** `conferenceData`.
Reuse a `requestId` only when retrying the same create.

### Notifications

Query param `sendUpdates`: `all` | `externalOnly` | `none`. Default is off
(insert) / treat as `none`. `sendNotifications` is deprecated. The API may
still send some mail.

### Event types

`eventType` is immutable after create. Default is `default`.

| Type | Where it works |
|------|----------------|
| `default` | any writable calendar |
| `outOfOffice` | **primary only** |
| `focusTime` | **primary only** |
| `workingLocation` | Workspace / feature-flagged accounts |
| `birthday` | special all-day annual; mostly read |
| `fromGmail` | created by Gmail; mostly read |

Consumer accounts reject `workingLocation` with a malformed-event error.
Do not probe OOO / focus on primary unless the user wants that block.

Other write fields worth knowing: `visibility` (`default` / `public` /
`private` / `confidential`), `transparency` (`opaque` blocks busy,
`transparent` does not), `status`, `colorId`, `recurrence`, `attendees[]`,
`guestsCanInviteOthers` / `Modify` / `SeeOtherGuests`.

## Patch vs update

```bash
# patch — send only the fields you are changing
gws calendar events patch \
  --params '{"calendarId":"primary","eventId":"EVENT_ID"}' \
  --json '{"summary":"New title"}'

# update — full replace; omitted fields are cleared
gws calendar events update \
  --params '{"calendarId":"primary","eventId":"EVENT_ID"}' \
  --json '{
    "summary":"New title",
    "start":{"dateTime":"2026-08-21T09:00:00-04:00","timeZone":"America/New_York"},
    "end":{"dateTime":"2026-08-21T09:15:00-04:00","timeZone":"America/New_York"}
  }'
```

Prefer **patch**. A summary-only `update` drops location, description,
visibility, attendees, and so on.

## Recurrence

Master event:

```json
"recurrence": ["RRULE:FREQ=WEEKLY;COUNT=4"]
```

Patch the **master** id to change the series. Patch an **instance** id
(`MASTER_20261214T140000Z`) to change one occurrence. Deleting an instance
cancels just that occurrence (`status: "cancelled"`); the rest of the series
stays.

There is no `thisAndFollowing` flag. To split a series, end the old RRULE
and insert a new master.

## quickAdd / import / move

```bash
gws calendar events quickAdd --params '{
  "calendarId":"primary",
  "text":"Lunch December 8 3pm-4pm"
}'

gws calendar events import --params '{"calendarId":"primary"}' --json '{
  "summary":"Imported copy",
  "iCalUID":"stable-uid@example.com",
  "start":{"dateTime":"2026-12-09T12:00:00-05:00","timeZone":"America/New_York"},
  "end":{"dateTime":"2026-12-09T12:30:00-05:00","timeZone":"America/New_York"}
}'

gws calendar events move --params '{
  "calendarId":"SOURCE_ID",
  "eventId":"EVENT_ID",
  "destination":"DEST_ID"
}'
```

`quickAdd` parses natural language into `start` / `end`; the stored summary
is usually the text with the time phrase stripped.

`import` needs `iCalUID`. It adds a private copy; it does not invite anyone.
Look it up later with `events.list` + `iCalUID`.

`move` changes the organizer calendar. Both calendars must be writable.

## Delete

```bash
gws calendar events delete --params '{
  "calendarId":"primary",
  "eventId":"EVENT_ID",
  "sendUpdates":"none"
}' --output ./empty.out
```

The event becomes `cancelled`. `events.get` still returns it. `showDeleted`
lists it. This is **not** Gmail Trash — guests who already have the invite
keep a cancelled copy if they were notified.

Empty DELETE bodies: gws treats the response as a download and writes
`download.html` in the **cwd** unless you pass `--output`. The path must
resolve inside the current directory or gws rejects the call **and does not
send the DELETE**.
