# Signetry Admission — GitHub Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Signetry%20Admission-purple?logo=github)](https://github.com/marketplace/actions/signetry-admission)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Latest release](https://img.shields.io/github/v/release/Signetry/action?sort=semver)](https://github.com/Signetry/action/releases)

**Govern any coding agent's change to your repository, and attach a signed receipt.**

Every pull request — no matter which agent opened it (Claude Code, Codex, Cursor,
Copilot, Devin, or a human) — is run through the [signetry-core](https://github.com/Signetry/core)
admission pipeline:

```
executable contract  →  untrusted-text quarantine  →  required checks  →
independent verifier  →  earned authority (0/1/2)  →  Ed25519-signed receipt
```

The action posts the verdict as a PR comment, uploads the signed receipt as an
artifact, and **fails the check** unless the change earns the authority you
require. Make it a *required status check* and nothing merges without a receipt.
`auto_merge` is always false — Signetry governs the agent; a human merges.

## Usage

```yaml
name: Signetry Admission
on:
  pull_request:
permissions:
  contents: read
  pull-requests: write
jobs:
  admit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
        with:
          ref: ${{ github.event.pull_request.head.sha }}
          fetch-depth: 0                       # base must be reachable for the diff
      - uses: Signetry/action@v1
        with:
          min-authority: "1"                   # 0 observe · 1 analyze · 2 branch-PR
          signing-key: ${{ secrets.SIGNETRY_SIGNING_KEY }}   # optional: stable signed receipts
```

Add a `.signetry/admission.yaml` to your repo to declare the contract (allowed and
forbidden paths, diff budget, required checks). Without one, a conservative
default applies. See the [signetry-core docs](https://github.com/Signetry/core).

### Also scan for vulnerabilities (SARIF → code scanning)

Turn on `scan` to run the SAST detection engine over the PR and upload SARIF to
GitHub code scanning alongside the admission verdict — deterministic, offline, and
free (7 languages, cross-file taint):

```yaml
permissions:
  contents: read
  pull-requests: write
  security-events: write        # to upload SARIF to code scanning
jobs:
  admit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
        with: { ref: ${{ github.event.pull_request.head.sha }}, fetch-depth: 0 }
      - uses: Signetry/action@v1
        with:
          scan: "true"
          scan-fail-on: "high"    # optional: fail the check on high+ findings
          min-authority: "1"
```


## Inputs

| Input | Default | Description |
|---|---|---|
| `mission` | review prompt | The bounded task the change claims to perform. |
| `min-authority` | `1` | Fail unless the change earns at least this level (0/1/2). |
| `agent` | `""` | Force `codex-cli` or `claude-code` to *re-run* the change. Blank governs the existing PR diff without invoking an agent. |
| `signing-key` | `""` | Base64 Ed25519 key (32+ bytes) for stable receipts. Falls back to a dev key (honestly flagged). |
| `require-sandbox` | `false` | Fail closed if code-executing checks (npm/pip install, go/cargo build) can't run in a real filesystem/network sandbox. |
| `scan` | `false` | Also run the **SAST detection engine** over the checkout and upload SARIF to code scanning (7 languages, cross-file taint, deterministic/offline; needs `signetry-core >= 0.5.0`). |
| `scan-fail-on` | `""` | With `scan`, fail the check if any finding is at/above this severity (`critical`/`high`/`medium`/`low`/`info`). Blank = report-only. |
| `signetry-version` | latest | Pin a specific `signetry-core` version **tag** installed from source (blank installs the latest hardened release). signetry-core is BUSL-1.1 (source-available, converting to Apache-2.0 on 2030-08-31) and is installed from its source repo, not PyPI. |
| `python-version` | `3.12` | Python to run on. |

## Outputs

| Output | Description |
|---|---|
| `authority-level` | The level the change earned (0/1/2). |
| `receipt-hash` | Canonical hash of the signed receipt. |
| `sarif-file` | Path to the SARIF file when `scan` is enabled (else empty). |
| `findings-count` | Number of detection findings when `scan` is enabled (else 0). |

## How it works

This action is Apache-2.0 and is a thin wrapper over `signetry-core` (BUSL-1.1,
source-available; installed from its source repo, not PyPI). It stages the
PR's change as a working-tree diff, runs `signetry admit`, and enforces the earned
authority. On Linux runners it installs bubblewrap so required checks run under a
real filesystem/network **sandbox** (the tier is recorded truthfully in every
receipt; it falls back to a lower tier only if the sandbox can't initialize). The
governance logic, contract, verifier, and receipts all live in
[signetry-core](https://github.com/Signetry/core).

Part of the [Signetry platform](https://github.com/Signetry/signetry) — see the umbrella for the full integration catalog and compatibility matrix.

## License

[Apache-2.0](LICENSE). Use it, fork it, ship it commercially — no strings.

This repository is part of Signetry's [open-core model](https://github.com/Signetry/signetry/blob/main/LICENSING.md):
the **integration surface is Apache-2.0** so anyone can add an agent, an editor, or a
CI adapter, while the engine ([`Signetry/core`](https://github.com/Signetry/core)) is
source-available under BUSL-1.1 and converts to Apache-2.0 on 2030-08-31.

Contributions are accepted under the [CLA](CLA.md) — it lets us move a well-built
adapter into the engine later without asking every contributor for permission again.
