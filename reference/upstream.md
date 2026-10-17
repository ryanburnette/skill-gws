# Upstream

- Repo: https://github.com/googleworkspace/cli
- npm: `@googleworkspace/cli`
- Note: **not** an officially supported Google product (per upstream README)
- Dynamic command surface from Google Discovery Service
- Agent skills ship upstream under their `skills/` (optional install)

Install options documented upstream: GitHub release binary, npm, cargo, brew, nix.

Pin the version for anything durable. Releases are irregular, not frequent:
a burst of releases can be followed by months with none (0.22.5, 2026-03-31,
was still the latest in October 2026), so a version going unchanged does not
mean anything is broken. Check for a newer one on a schedule rather than
assuming:

```sh
gws --version | head -1                        # installed
npm view @googleworkspace/cli version          # latest published
```
