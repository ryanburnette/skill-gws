# client_secret.json format

gws expects standard Google **installed/Desktop** download shape:

```json
{
  "installed": {
    "client_id": "PROJECTNUM-xxxx.apps.googleusercontent.com",
    "project_id": "YOUR_PROJECT_ID",
    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://oauth2.googleapis.com/token",
    "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
    "client_secret": "GOCSPX-…",
    "redirect_uris": ["http://localhost"]
  }
}
```

Default path: `$HOME/.config/gws/client_secret.json` (chmod 600).

Env alternative:

```bash
export GOOGLE_WORKSPACE_CLI_CLIENT_ID='…'
export GOOGLE_WORKSPACE_CLI_CLIENT_SECRET='…'
```

Do not put real values in this git repo.
