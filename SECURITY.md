# Security Policy

Lafiya's contracts are **pre-alpha, unaudited, and targeted at Stellar
testnet** — but they anchor a health-adjacent trust layer, so
vulnerabilities matter even before mainnet. Please report them
responsibly.

## Reporting a vulnerability

**Do not open a public GitHub issue, PR, or discussion for a security
report.** Public disclosure before a fix exists puts users at risk.

Instead, report privately via GitHub's private vulnerability reporting:
open this repository's **Security** tab → **Advisories** →
[**Report a vulnerability**](../../security/advisories/new).

Include, where possible:

- A description of the vulnerability and its impact (e.g. forged
  attestation, allowlist bypass, auth-confusion in a cross-contract
  call).
- Steps to reproduce or a proof of concept (a failing test under
  `contracts/*/src/test.rs` is ideal).
- The contract(s) affected: `attester-registry` and/or
  `attestation-registry`.
- Any suggested mitigation.

> Never include real personal or health data in a report. The contracts
> store only non-reversible hashes by design — keep reports the same
> way. See the privacy note in [README.md](README.md).

## What to expect

- **Acknowledgement** of your report within a few days.
- **Triage and severity assessment**, with follow-up questions routed
  back through the private advisory thread.
- **A fix and coordinated disclosure**: we aim to patch before any
  public detail is published, and we credit reporters (unless you'd
  rather stay anonymous).

## Scope

In scope:

- The Soroban contracts in this repository (`contracts/attester-registry`,
  `contracts/attestation-registry`) — authorization, initialization,
  storage, events, and the cross-contract call between them.
- Build/CI configuration in this repo where a defect could cause
  malicious artifacts to be trusted.

Out of scope:

- Vulnerabilities in the Stellar network, the Soroban SDK, or Rust
  toolchain themselves — report those upstream.
- The `lafiya-web` app and other sibling repositories (they have, or
  will have, their own policies); see the README's
  [Lafiya Organization](README.md#lafiya-organization) section.

## Supported versions

The project is pre-release (`0.x`). Only the latest commit on `main` is
supported with security fixes; there are no maintained release branches
yet.

## After a fix

Once a vulnerability is fixed, details may be published via a GitHub
security advisory and noted in [CHANGELOG.md](CHANGELOG.md). General
contributions (non-security) follow the workflow in
[CONTRIBUTING.md](CONTRIBUTING.md).

## CI supply-chain hardening

The CI pipeline builds the wasm that becomes trusted contract code, so it is
part of the trust base.

- **Pinned actions.** Every `uses:` reference is pinned to a full commit SHA
  with the release in a trailing comment (`uses: actions/checkout@<sha> # v7.0.1`).
  Dependabot's `github-actions` ecosystem (`.github/dependabot.yml`) opens PRs
  that bump the SHA and the comment together. Never reference an action by tag
  or branch.
- **Least-privilege tokens.** Every workflow declares top-level
  `permissions: contents: read`. Jobs raise it only where needed:
  `docs.yml` (`contents: write`, to push `gh-pages`), `stale.yml`
  (`issues`/`pull-requests: write`), and `scorecard.yml` / `security-scan.yml`
  (`security-events: write`, plus `id-token: write` for Scorecard publishing).
- **Verified downloads.** Binaries fetched with `curl` (stellar-cli in
  `smoke-test.yml`, gitleaks in `security-scan.yml`) are checked against a
  pinned SHA-256 with `sha256sum -c`. Bump the URL and checksum together.
- **Runner hardening.** The release (`release-manifest.yml`) and deploy
  (`docs.yml`) jobs run `step-security/harden-runner` in `egress-policy: audit`.
  After reviewing the observed endpoints from a few runs, switch to
  `egress-policy: block` with an `allowed-endpoints` list.
- **OpenSSF Scorecard** (`scorecard.yml`) runs weekly and on every push to
  `main`, uploads results to code scanning, and publishes the README badge.
  Baseline score: *not yet published* (no Scorecard run existed before this
  workflow). Record the first run's score here as the "before" value and the
  score after the fixes it reports as the "after" value.

### Recommended ruleset for `main`

Maintainers should apply these via **Settings → Rules → Rulesets** (they
cannot be set from a pull request):

| Setting | Value |
| --- | --- |
| Restrict deletions / block force pushes | On |
| Require a pull request before merging | On, 1 approval |
| Require review from Code Owners | On (see `.github/CODEOWNERS`) |
| Dismiss stale approvals on new commits | On |
| Require status checks to pass | `Shell lint (smoke-test.sh)`, `Format`, `Clippy`, `Test`, `Build (wasm32v1-none)`, `docs`, `CodeQL (*)`, `Semgrep (Soroban rules)`, `gitleaks` |
| Require branches to be up to date before merging | On |
| Require signed commits | Optional (recommended once all maintainers sign) |
| Bypass list | Empty (admins included) |

**Would this have blocked the `lafiya-cli` build breakage?** Only if the
failing job is a *required* check and branches must be up to date. A
breakage that passes on a stale PR branch but fails after merging with a
newer `main` is exactly what "require branches to be up to date" catches;
without required checks, a red CI run does not prevent merging. Enabling
both settings above closes that gap.
