# Setup lessons learned

1. **`gws auth setup` is not fully unattended** without a browser TTY and with
   OAuth client APIs removed — expect manual Console steps for the Desktop client.

2. **IAP `oauth-brands` / old automated client creation** — deprecated/shut down;
   personal projects hit “must belong to an organization” or explicit manual instructions.

3. **IAM `locations/global/oauthClients`** creates a different kind of client id
   (not `*.apps.googleusercontent.com`). Useless for stock gws Desktop login.

4. **Testing mode scope limit (~25)** — do not use full recommended scope pack
   for consumer Gmail; use `-s gmail,drive,calendar,sheets,docs` (or tighter).
   Workspace-only scopes in the big pack can 400 on `@gmail.com`.

5. **File keyring** is effectively required on headless Linux
   (`GOOGLE_WORKSPACE_CLI_KEYRING_BACKEND=file`).

6. **Callback paste** is the reliable headless Desktop pattern; public HTTP
   reverse proxy does not help loopback redirect_uris.

7. **gcloud and gws are separate auth piles** — gcloud login ≠ gws Gmail scopes.

8. **Multi-account** — credential precedence is documented (`TOKEN` >
   `CREDENTIALS_FILE` > login store); named profiles are not a first-class
   feature. Separate config dirs via `GOOGLE_WORKSPACE_CLI_CONFIG_DIR` is a
   practical approach; domain-wide delegation helps Workspace only.

9. **Internal consent removes the testing-mode pain entirely.** The ~25 scope
   cap in lesson 4, the test-user list, and the "Google hasn't verified this
   app" interstitial are all artifacts of *External* + testing mode. A project
   created **inside a Workspace organization** can set the consent screen
   Audience to **Internal**, and none of the three apply. Two conditions: the
   project must be in the org (at creation, the Location field defaults to *No
   organization* — get it right or Internal is greyed out later), and there is
   no reason to register scopes on the Data Access tab, since a Desktop client
   requests scopes at login.

10. **`<api>:<version>` is not an escape hatch for unlisted APIs**, despite what
    the unknown-service error says. The suffix only overrides the version of a
    service gws already knows (`gws drive:v3 files list`). An unknown service
    name fails first, as a local `validationError`, so another credential will
    not help.

11. **Enabling an API and granting a scope are separate steps.** Enabling in the
    Console is free, instant, and grants nothing by itself — the token's scopes
    decide what is callable. So be generous when enabling APIs, but remember
    that adding a service later needs the API enabled *and* a re-login to pick
    up its scope. Scopes freeze at login.

12. **`--scopes` replaces the scope set; it does not add to it.** Pass the union
    of current and new scopes, or the surfaces you already had stop working.
    Read the current set from `gws auth status` first:

    ```sh
    gws auth status | jq -r '.scopes[]'

    B=https://www.googleapis.com/auth
    gws auth login --scopes "openid,email,profile,$B/drive,$B/NEW_SCOPE"
    ```

    Verify from the login output, which echoes the granted `scopes` array, and
    then smoke-test one call per surface you depended on before. A partial
    re-login is silent until something fails days later.

    Two scope counts disagree legitimately. `gws auth status` reported 13 while
    the token endpoint reported 10 for the same credential, because `openid`,
    `email`, and `profile` collapse into the `userinfo.*` URLs. Compare scope
    *names*, not counts.

13. **`gws auth export` returns no access token** — the fields are
    `client_id`, `client_secret`, `refresh_token`, `type`. To call an API that
    gws has no surface for, do the refresh grant yourself:

    ```sh
    curl -sS -X POST https://oauth2.googleapis.com/token \
      -d client_id="$CID" -d client_secret="$CSEC" \
      -d refresh_token="$RTOK" -d grant_type=refresh_token
    ```

    The response includes `access_token` and the granted `scope` string, which
    is a quick way to confirm what a credential actually carries. Keep all of it
    in memory; never write it to a file or a log.

    Reaching for the field that isn't there fails misleadingly: `jq -r
    '.access_token'` prints the string `null`, which survives a `test -n` guard
    and is sent as `Authorization: Bearer null`, and Google's reply reads as an
    expired token.
