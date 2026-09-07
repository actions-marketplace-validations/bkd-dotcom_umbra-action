# Contributing

Thanks for wanting to improve **Signetry Admission**. This repository is the
GitHub Action — the integration surface — and it is
[Apache-2.0](LICENSE): use it, fork it, ship it commercially, no strings.

## What the licence lets you do

Apache-2.0 gives you a patent grant and the right to use, copy, modify,
distribute, and commercialize this Action, including in closed-source and
commercial products. You do not need our permission and you do not owe us
anything. Keep the `LICENSE` and the attribution notices when you redistribute,
and note your changes — that is the whole obligation.

This repo is part of Signetry's
[open-core model](https://github.com/Signetry/signetry/blob/main/LICENSING.md):
every integration (this Action, the editor and agent plugins, the pre-commit
guard, the eval suite) is Apache-2.0, while the engine
([`Signetry/core`](https://github.com/Signetry/core)) is source-available under
BUSL-1.1 and converts to Apache-2.0 on 2030-08-31.

## The CLA still applies — and why

A PR **cannot be merged** until you sign the
[Contributor License Agreement](CLA.md). It is enforced by a bot: when you open a
pull request, the **CLA Assistant** check asks you to reply on the PR with exactly:

```
I have read the CLA Document and I hereby sign the CLA
```

Your acceptance is recorded in `signatures/cla.json`.

The CLA is not about withholding rights from you — Apache-2.0 already grants you
everything above, and signing does not take it away. It exists because code moves
across the open-core line. A well-built adapter that starts here as Apache-2.0
may later belong in the BUSL-1.1 engine, and Signetry needs the relicensing
rights to move it without tracking down every past contributor for permission.
It also lets us dual-license and defend the project if that is ever necessary.

## Getting started

There is no build step and no compiled artifact. The Action is a **composite
action defined entirely in [`action.yml`](action.yml)** — a series of `shell: bash`
steps that install `signetry-core` from its source repo and run
`signetry admit` / `signetry scan` / `signetry comment`. Editing this repo means
editing that YAML (or the workflows in `.github/workflows/`) and the docs.

To exercise a change, point a workflow in a scratch repository at your branch:

```yaml
      - uses: Signetry/action@my-branch    # or your-fork/action@my-branch
        with:
          min-authority: "1"
```

…then open a PR in that scratch repo and read the run log, the verdict comment,
and the uploaded receipt artifact.

Two workflows run on every PR here:

- **CLA** (`.github/workflows/cla.yml`) — the signature gate described above.
- **Reviewer** (`.github/workflows/reviewer.yml` + `reviewer-comment.yml`) — an
  advisory `signetry-reviewer` pass that posts one recommendation comment. It is
  advisory only: it never merges and never fails the PR.

### Where a change belongs

- **This repo** — action inputs and outputs, the composite steps, SARIF upload,
  the PR comment plumbing, runner/sandbox setup, docs.
- **[`Signetry/core`](https://github.com/Signetry/core)** — the governance logic
  itself: the contract, injection quarantine, required checks, the independent
  verifier, earned authority, and receipt signing. If the verdict is wrong, the
  bug is almost certainly there, not here.

### Things to keep in mind

`action.yml` is a security-critical surface, and `.github/CODEOWNERS` routes it
for review accordingly:

- Never interpolate `${{ inputs.* }}` or `${{ github.event.* }}` directly into a
  `run:` body — pass it through `env:` and read the variable, as the existing
  steps do. That was a real script-injection vulnerability in `v0.1.0`–`v0.1.2`
  (see [SECURITY.md](SECURITY.md)).
- Do not add `pull_request_target` with a checkout of PR head, and do not give a
  job that executes PR code a writable token. The split between `reviewer.yml`
  (untrusted code, read-only) and `reviewer-comment.yml` (trusted, writable, never
  checks out PR code) is deliberate — the header comments in both files explain it.
- Fail closed, not open. A step that cannot verify something should refuse, not
  wave the change through.
- Update [`CHANGELOG.md`](CHANGELOG.md) under `## [Unreleased]` for anything a
  user would notice.

Found a vulnerability? Do not open a public issue — use
[private reporting](https://github.com/Signetry/action/security/advisories/new).
See [SECURITY.md](SECURITY.md).

By participating you agree to the [Code of Conduct](CODE_OF_CONDUCT.md).

## Credit

Contributors are **acknowledged** in [CONTRIBUTORS.md](CONTRIBUTORS.md), the Git
history, and release notes. See the "Recognition of Contributors" clause in
[CLA.md](CLA.md).
