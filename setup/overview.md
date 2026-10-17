# Setup overview (generic)

## Goal

Install `gws`, create or reuse a GCP project, enable Workspace APIs, configure a
Desktop OAuth client, and complete `gws auth login` on a (possibly headless) host.

## Typical path

1. Install Node (or use a prebuilt `gws` binary — see [reference/upstream.md](../reference/upstream.md)).
2. `npm install -g @googleworkspace/cli` (pin version for durable automation).
3. Install gcloud if you will manage the project/APIs from the CLI.
4. `gcloud auth login` (headless: `--no-launch-browser`).
5. Create/select a GCP project; enable Gmail/Drive/Calendar/… APIs as needed.
6. **Manual** Google Auth Platform setup in Console (consent + **Desktop** client).
   Automated OAuth client creation for classic Desktop clients is largely gone.
7. Write Desktop client JSON to `$HOME/.config/gws/client_secret.json` (mode 600).
8. `GOOGLE_WORKSPACE_CLI_KEYRING_BACKEND=file gws auth login -s gmail,drive,calendar,sheets,docs`
   (narrower scopes if possible).
9. Complete headless callback — see [headless-oauth.md](./headless-oauth.md).
10. Smoke: Gmail profile, Drive files list, Calendar calendarList.

## Related

- [headless-oauth.md](./headless-oauth.md)
- [oauth-client-json.md](./oauth-client-json.md)
- [lessons.md](./lessons.md)

Host-specific project ids, checklists, and “what we did on date X” belong in the
project/host repo, not this skill.
