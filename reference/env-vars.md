# Environment variables (gws)

| Variable | Purpose |
|----------|---------|
| `GOOGLE_WORKSPACE_CLI_KEYRING_BACKEND=file` | Headless encrypted creds (typical on Linux servers) |
| `GOOGLE_WORKSPACE_CLI_TOKEN` | Pre-baked access token (highest precedence) |
| `GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE` | Path to SA or authorized_user JSON |
| `GOOGLE_WORKSPACE_CLI_CLIENT_ID` | OAuth client id |
| `GOOGLE_WORKSPACE_CLI_CLIENT_SECRET` | OAuth client secret |
| `GOOGLE_WORKSPACE_CLI_CONFIG_DIR` | Override config dir (default `$HOME/.config/gws`) |
| `GOOGLE_WORKSPACE_PROJECT_ID` | Project override for quota/helpers |
| `GWS_SANITIZE_TEMPLATE` | Model Armor template (optional) |

Also supports `.env` via dotenv for gws.

## Separate credential directories

`GOOGLE_WORKSPACE_CLI_CONFIG_DIR` selects a configuration directory, not an
account by email. Separate directories can keep different login credentials
apart. Apply the intended directory to both `gws auth login` and subsequent
commands; changing it only for login does not select those credentials later.
Verify the intended account during browser consent and with a read-only API
call afterward.

Check inherited environment variables and `.env` files first:
`GOOGLE_WORKSPACE_CLI_TOKEN` and `GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE` take
precedence over stored encrypted login credentials. A directory override
alone therefore does not guarantee which identity a command uses. Keep
credential files, encryption keys, and exported tokens out of logs and git.

Source: upstream [authentication precedence and environment variables](https://github.com/googleworkspace/cli/blob/8e945eed02a8230af122ec6247cc218d28c1f1ce/README.md#authentication).
