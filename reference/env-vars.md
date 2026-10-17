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
