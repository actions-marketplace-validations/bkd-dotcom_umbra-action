# Security Policy

Signetry Admission is a security tool; we hold its own security to a high bar.

## Supported versions

Pin the Action to **`@v1`** (moving major tag — always the latest hardened
release) or a specific `@v0.1.3+` tag.

| Version | Supported |
|---|---|
| `v1` / `>= v0.1.3` | ✅ current |
| `v0.1.0`–`v0.1.2` (exact pins) | ⚠️ **upgrade now** |

`v0.1.0`–`v0.1.2` contained a GitHub Actions **script-injection** sink (action
inputs interpolated into a shell body) and a fail-open PR-staging path. These are
fixed in `v0.1.3`. If you pinned an exact old tag, move to `@v1`.

## Reporting a vulnerability

Do **not** open a public issue. Use private reporting:
**https://github.com/Signetry/action/security/advisories/new**
(or report core issues at
https://github.com/Signetry/core/security/advisories/new).

## Safe usage

- Grant the workflow the least privilege it needs: `contents: read` and
  `pull-requests: write` (for the verdict comment). Do not pass extra secrets to
  the action unless you intend the checks to use them.
- Never plumb attacker-controlled text (PR title, branch, issue body) into the
  `mission` input; the action passes inputs via environment and validates them,
  but least privilege is still the right posture.
- Use `signing-key: ${{ secrets.SIGNETRY_SIGNING_KEY }}` for receipts that verify
  against your pinned key; without it, receipts use a dev key and are flagged
  `key_ephemeral`.
- For fail-closed check isolation set `require-sandbox: true`.
