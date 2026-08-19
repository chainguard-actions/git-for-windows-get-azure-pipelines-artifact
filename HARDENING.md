<!-- markdownlint-disable -->

# Hardening Report: git-for-windows--get-azure-pipelines-artifact/v0.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **git-for-windows--get-azure-pipelines-artifact/v0.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files reference GitHub Actions using mutable version tags instead of full 40-character commit SHA pins, making them vulnerable to supply-chain attacks if the referenced tags are moved or compromised.

- .github/workflows/npm-run-package.yml: `uses: actions/checkout@v2`
- .github/workflows/release-tag.yml: `uses: actions/checkout@v2`, `uses: actions/github-script@v3` (appears twice)
- .github/workflows/test.yml: `uses: actions/checkout@v2` (appears three times), `uses: actions/upload-artifact@v2`

All should be pinned to their full SHA digest, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v2`.

Locations:

- `.github/workflows/npm-run-package.yml:24`
- `.github/workflows/release-tag.yml:13`
- `.github/workflows/release-tag.yml:16`
- `.github/workflows/release-tag.yml:60`
- `.github/workflows/test.yml:15`
- `.github/workflows/test.yml:32`
- `.github/workflows/test.yml:37`
- `.github/workflows/test.yml:49`
- `.github/workflows/test.yml:50`

### script-injection (severity: high)

Sub-rule (a): The 'commit & push' step in npm-run-package.yml directly interpolates `${{github.actor}}` inside a `run:` shell command. The `github.actor` value is substituted by the YAML template engine before the shell processes the string, allowing an attacker with a specially crafted username to inject arbitrary shell commands. The offending lines are:
  `git config user.name "${{github.actor}}" &&`
  `git config user.email "${{github.actor}}@users.noreply.github.com" &&`

Fix: move the value into an `env:` variable and double-quote the shell expansion, e.g.:
  env:
    ACTOR: ${{ github.actor }}
  run: git config user.name "$ACTOR"

Locations:

- `.github/workflows/npm-run-package.yml:41`
- `.github/workflows/npm-run-package.yml:42`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions, violating the principle of least privilege.

- .github/workflows/npm-run-package.yml: triggered by `pull_request_target` (a high-privilege trigger) with no permissions block at all.
- .github/workflows/test.yml: no permissions block at any level.

Locations:

- `.github/workflows/npm-run-package.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across the three workflow files:

1. unpinned-uses: Pinned all mutable action tags to full SHA digests with tag comments:
   - actions/checkout@v2 → @0717577d45739eb3c851188b29f50ed6c0b2194e # v2 (5 occurrences across all 3 files)
   - actions/github-script@v3 → @ffc2c79a5b2490bd33e0a41c1de74b877714d736 # v3 (2 occurrences in release-tag.yml)
   - actions/upload-artifact@v2 → @82c141cc518b40d92cc801eee768e7aafc9c2fa2 # v2 (1 occurrence in test.yml)

2. script-injection: In npm-run-package.yml 'commit & push' step, moved ${{github.actor}} out of the run: shell string into an env: block as ACTOR, then referenced it as $ACTOR in the shell script.

3. missing-permissions: Added top-level permissions block to npm-run-package.yml (contents: write, needed to push commits) and test.yml (contents: read, minimal needed for checkout). release-tag.yml already had permissions: contents: write.

