---
name: gws
description: >-
  Google Workspace CLI (gws) — living knowledge from using the tool. Install,
  OAuth, Gmail, Calendar, Drive, and other non-admin surfaces as we learn them.
  Use when running or documenting gws on any host. No personal or host-specific
  facts here. Admin SDK notes do not belong in this skill.
---

# skill-gws

Living knowledge for **[gws](https://github.com/googleworkspace/cli)**
(`@googleworkspace/cli`). Not an officially supported Google product.

Write down what we learn while using the CLI. Improve it as we go. Cover every
surface gws can reach — Gmail, Calendar, Drive, Docs, Sheets, and whatever
else shows up — not only the topics already filed here. Gmail is still the
deepest set of notes; Drive has sharing / shared drives; Calendar has agenda,
events, and ACL notes. Admin SDK behavior is out of scope for this skill. The
other surfaces should catch up as we use them.

Keep this **portable and non-identifying**. No personal names, emails, machine
names, GCP project ids, or one-off ops logs. Those belong in the consuming
project.

## When to load what

| Need | Start here |
|------|------------|
| Safety defaults while automating | below |
| Headless / Desktop OAuth | [setup/headless-oauth.md](./setup/headless-oauth.md) |
| OAuth client JSON shape | [setup/oauth-client-json.md](./setup/oauth-client-json.md) |
| Setup footguns | [setup/lessons.md](./setup/lessons.md) |
| **Gmail index (start here)** | [gmail/overview.md](./gmail/overview.md) |
| Read / triage / get formats | [gmail/reading.md](./gmail/reading.md) |
| Primary / Promotions / Social tabs | [gmail/categories-and-tabs.md](./gmail/categories-and-tabs.md) |
| Archive, labels, mark read, batch | [gmail/organize.md](./gmail/organize.md) |
| Send / reply / drafts | [gmail/compose.md](./gmail/compose.md) |
| Download attachments | [gmail/attachments.md](./gmail/attachments.md) |
| History sync + watch | [gmail/history-and-watch.md](./gmail/history-and-watch.md) |
| Filters and account settings | [gmail/settings-and-filters.md](./gmail/settings-and-filters.md) |
| Trash vs delete | [gmail/destructive-ops.md](./gmail/destructive-ops.md) |
| Helpers vs raw / risk ladder | [gmail/helpers-and-raw.md](./gmail/helpers-and-raw.md) |
| Gmail method map | [reference/gmail-api-map.md](./reference/gmail-api-map.md) |
| Query recipes | [reference/gmail-queries.md](./reference/gmail-queries.md) |
| Bulk trash procedure | [runbooks/trash-by-gmail-query.md](./runbooks/trash-by-gmail-query.md) |
| Re-auth (generic) | [runbooks/headless-reauth.md](./runbooks/headless-reauth.md) |
| **Drive index** | [drive/overview.md](./drive/overview.md) |
| Share files / folders / drives | [drive/sharing.md](./drive/sharing.md) |
| My Drive vs shared drives | [drive/my-drive-vs-shared-drives.md](./drive/my-drive-vs-shared-drives.md) |
| **Calendar index** | [calendar/overview.md](./calendar/overview.md) |
| Agenda, list/get, free/busy, settings | [calendar/reading.md](./calendar/reading.md) |
| Create / patch / Meet / recurrence | [calendar/events.md](./calendar/events.md) |
| Calendars, sidebar, ACL | [calendar/calendars-and-sharing.md](./calendar/calendars-and-sharing.md) |
| Calendar method map | [reference/calendar-api-map.md](./reference/calendar-api-map.md) |
| Cheatsheet / env vars | [reference/](./reference/) |

## Safety defaults (portable)

- Read-only Gmail/Drive/Calendar unless the user asks to mutate.
- Prefer **Trash** over permanent `delete` / `batchDelete`.
- Before bulk mutate: real count via pagination + metadata sample + matching query.
- Do **not** trust `resultSizeEstimate`; use `labels.get` or paginated id counts.
- UI tabs ≠ folders; use `in:inbox category:…` for tab semantics.
- `gmail.modify` can **send** mail — not “organize only.”
- Testing-mode OAuth: limited scopes (`-s gmail,drive,…`); avoid full recommended set on consumer accounts (~25 scope cap; Workspace-only scopes break personal login).
- Large mailboxes: always bound `maxResults` (≤500) / `q`; avoid casual `--page-all`.
- List existing **filters** before automating rules that might conflict.
- Never `calendars.clear` (wipes primary). Prefer a throwaway secondary calendar for Calendar experiments, then `calendars.delete` it.
- Never commit or print secrets (`client_secret.json`, refresh material, `gws auth export --unmasked`, encryption keys, access tokens).

## Knowledge hygiene for this skill

1. Capture **reusable** gws knowledge here as you go (commands, API quirks, query recipes, footguns).
2. Prefer small topic files. Add directories when a new surface has enough notes to deserve one (`calendar/`, `drive/`, …). Admin SDK notes do not go here.
3. Keep examples generic (`user@example.com`, `PROJECT_ID`, `$HOME`). Never use a real person, account, or host as an example.
4. After meaningful changes: commit and push this repo per the host’s git policy.
5. **Host/project facts** (which machine, which Google account, which GCP project, daily ops log, product plans) stay in the consuming project, not here.

## Companion note

Scripting style is not defined by this skill. Use the host’s shell, JS, prose,
and git skills when those are loaded.
