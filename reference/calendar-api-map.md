# Calendar API map (via gws)

Invocation: `gws calendar <resource> <method> --params '{…}' [--json '{…}']`  
Helpers: `gws calendar +agenda`, `gws calendar +insert`  
Schema: `gws schema calendar.events.list` or `gws schema calendar.Event`

## Helpers

| Command | Risk | Notes |
|---------|------|--------|
| `+agenda` | read | All calendars; windows in [../calendar/reading.md](../calendar/reading.md) |
| `+insert` | mutate | Default calendar `primary`; `--meet` needs conference v1 |

Workflow (also read Calendar): `+weekly-digest`, `+standup-report`,
`+meeting-prep`.

## events

| Method | Risk | Notes |
|--------|------|--------|
| `list` | read | `calendarId` required; RFC3339 offsets; `orderBy=startTime` needs `singleEvents` |
| `get` | read | Cancelled events still return |
| `instances` | read | Occurrences of one series |
| `insert` | mutate | `conferenceDataVersion=1` to create Meet |
| `patch` | mutate | Partial; prefer over update |
| `update` | mutate | Full replace; omitted fields drop |
| `quickAdd` | mutate | Natural-language `text` |
| `import` | mutate | Needs `iCalUID`; private copy |
| `move` | mutate | Changes organizer calendar |
| `delete` | mutate | Sets `cancelled`; empty body → `--output` |
| `watch` | push config | Needs webhook |

`sendUpdates`: `all` \| `externalOnly` \| `none` (default off).

## calendarList

| Method | Risk | Notes |
|--------|------|--------|
| `list` / `get` | read | Hidden omitted unless `showHidden` |
| `insert` | mutate | Subscribe to existing id |
| `patch` / `update` | mutate | Color, hidden, selected, override title |
| `delete` | mutate | Unsubscribe only |
| `watch` | push config | |

## calendars

| Method | Risk | Notes |
|--------|------|--------|
| `get` | read | Metadata |
| `insert` | mutate | Secondary; caller is data owner |
| `patch` / `update` | mutate | Title, description, tz |
| `delete` | **permanent** | Secondary calendar + events |
| `clear` | **wipes primary** | Never unless explicitly asked |
| `transferOwnership` | admin | Workspace only |

## acl

| Method | Risk | Notes |
|--------|------|--------|
| `list` / `get` | read | |
| `insert` / `patch` / `update` | share | `sendNotifications` defaults **true** |
| `delete` | mutate | |
| `watch` | push config | |

## Other

| Resource | Method | Risk |
|----------|--------|------|
| `freebusy` | `query` | read — POST `--json` with `timeMin`/`timeMax`/`items` |
| `settings` | `list` / `get` | read |
| `colors` | `get` | read |
| `channels` | `stop` | ends a watch |

## Common list params (`events.list`)

`calendarId` · `timeMin` · `timeMax` · `timeZone` · `singleEvents` ·
`orderBy` · `q` · `eventTypes` · `iCalUID` · `showDeleted` ·
`showHiddenInvitations` · `maxResults` · `pageToken` · `syncToken` ·
`updatedMin` · `maxAttendees` · `fields`
