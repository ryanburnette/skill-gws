# Runbook: re-auth gws (headless Desktop)

1. Ensure Desktop `client_secret.json` is still present and the account is still a test user (if the app is in Testing).
2. `export GOOGLE_WORKSPACE_CLI_KEYRING_BACKEND=file`
3. `gws auth login -s gmail,drive,calendar,sheets,docs` (or the scopes you actually need)
4. User completes browser consent; paste `http://localhost:PORT/?code=…` URL
5. `curl` callback on the gws host (or SSH `-L`) — see [setup/headless-oauth.md](../setup/headless-oauth.md)
6. `gws auth status` → `token_valid`, expected user email
7. Smoke: `gws gmail users getProfile --params '{"userId":"me"}'`
8. Record that re-auth happened in the **host/project** log (no secrets)
