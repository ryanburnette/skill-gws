# skill-gws

Living knowledge for the **Google Workspace CLI (`gws`)**.

We write down what we learn while using the tool, across every surface it
reaches, and keep improving the notes. Portable and non-identifying: no
personal names, emails, machine names, or GCP project ids. Those stay in the
consuming project.

## Layout

```
gws/   # often cloned as ~/.pi/agent/skills/gws for pi
  SKILL.md            Skill entry, purpose, safety defaults
  setup/              Install/OAuth patterns (generic)
  gmail/              Deepest surface so far
  drive/              Sharing, My Drive vs shared drives
  calendar/           Agenda, events, calendars, ACL
  runbooks/           Reusable procedures
  reference/          API maps, queries, cheatsheet, env vars, upstream
```

Add a directory when another non-admin surface has enough durable notes.
Admin SDK notes do not belong in this skill.
Gmail starts at [`gmail/overview.md`](./gmail/overview.md). Drive starts at
[`drive/overview.md`](./drive/overview.md). Calendar starts at
[`calendar/overview.md`](./calendar/overview.md).

## Install for pi

```bash
mkdir -p ~/.pi/agent/skills
git clone <this-repo-url> ~/.pi/agent/skills/gws
```

## Git

```bash
cd ~/.pi/agent/skills/gws   # or your clone path
git add path/to/file && git commit -m "…"
git push origin main
```

## Secrets

Never store OAuth secrets or tokens here. Live material stays in
`$HOME/.config/gws/` (or the path set by `GOOGLE_WORKSPACE_CLI_CONFIG_DIR`).
