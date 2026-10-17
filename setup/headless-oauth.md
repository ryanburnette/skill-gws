# Headless OAuth for gws

## Why public ingress does not fix Desktop login

Desktop clients redirect only to **loopback** (`http://localhost:PORT` /
`127.0.0.1`). Google will not redirect to a public host URL unless you create a
**Web** application client with that redirect URI. Prefer Desktop + callback
paste, or SSH `-L` port forward.

## File keyring

```bash
export GOOGLE_WORKSPACE_CLI_KEYRING_BACKEND=file
```

Without this, headless Linux hosts often fail storing encrypted credentials.
Key file: `$HOME/.config/gws/.encryption_key` (unless `GOOGLE_WORKSPACE_CLI_CONFIG_DIR` overrides).

## gws login recipe (proven)

1. Start (limit services — testing apps ~25 scope cap; full `recommended` often breaks on consumer accounts):

   ```bash
   gws auth login -s gmail,drive,calendar,sheets,docs
   ```

2. Note printed URL and the `redirect_uri=http://localhost:PORT`.
   Listener binds `127.0.0.1:PORT` on the machine running gws.

3. User opens URL as the intended Google account:
   - App in testing → account must be a **test user**
   - “Google hasn’t verified this app” → Continue
   - Approve scopes

4. Browser goes to `http://localhost:PORT/?code=...` on **the user’s machine**
   (connection failed page). User copies **full address-bar URL**.

5. On the gws host, deliver callback to the listener:

   ```bash
   curl -sS 'http://localhost:PORT/?iss=...&code=...&scope=...'
   ```

   Expect HTTP 200 body like “You may now close this window” and gws JSON
   `status: success` with `credentials.enc` written.

### Workspace domain notes

The recipe above is unchanged on a Workspace domain, but two details are worth
knowing.

**Scope expansion.** `-s gmail,drive,calendar,sheets,docs,reports` requests
exactly these ten scopes:

```
drive  spreadsheets  gmail.modify  calendar  documents
admin.reports.audit.readonly  admin.reports.usage.readonly
openid  userinfo.email  userinfo.profile
```

Note `gmail.modify`, not `https://mail.google.com/`. gws picks the narrower
scope, so **permanent message deletion is not grantable** through this set —
a useful safety floor that comes for free. Both admin scopes are read-only.

**The returned callback carries extra params.** Google appends `iss`,
`authuser`, `hd` (the Workspace domain), and `prompt` alongside `code` and
`scope`. Deliver the whole address-bar URL verbatim to the listener; the extra
params are harmless. Single-quote it in the shell — the `scope` value contains
`%20`-separated URLs and the `code` contains `/`.

Expect `status: success` plus `account`, the granted `scopes` array, and
`credentials.enc`. Verify with a read-only call per surface rather than trusting
the login output alone.

### Optional: SSH port forward

```bash
ssh -L PORT:127.0.0.1:PORT user@remote
```

Then the browser callback hits the forwarded port and step 5 is automatic.

## gcloud headless login

```bash
gcloud auth login --no-launch-browser
```

Prints Google URL; user returns a **verification code** (not a localhost URL).
Can be driven with a waiting process + stdin FIFO if the agent cannot be TTY-interactive.

## Auth precedence (gws)

1. `GOOGLE_WORKSPACE_CLI_TOKEN`
2. `GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE`
3. Encrypted credentials from `gws auth login`
4. Plaintext `$HOME/.config/gws/credentials.json`

## Export (sensitive)

```bash
gws auth export --unmasked   # full secrets to stdout — do not log/commit
```

Use only to mint short-lived API access tokens in memory for bulk jobs.
