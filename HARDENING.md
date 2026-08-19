<!-- markdownlint-disable -->

# Hardening Report: git-for-windows--get-azure-pipelines-artifact/v0.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **git-for-windows--get-azure-pipelines-artifact/v0.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable version tags (e.g. @v2, @v3) rather than immutable 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the upstream action tag is moved or compromised. Affected references:
- .github/workflows/release-tag.yml: `actions/checkout@v2` (line 16), `actions/github-script@v3` (line 20), `actions/github-script@v3` (line 64)
- .github/workflows/test.yml: `actions/checkout@v2` (lines 16, 33, 44, 45)

Locations:

- `.github/workflows/release-tag.yml:16`
- `.github/workflows/release-tag.yml:20`
- `.github/workflows/release-tag.yml:64`
- `.github/workflows/test.yml:16`
- `.github/workflows/test.yml:33`
- `.github/workflows/test.yml:44`
- `.github/workflows/test.yml:45`

### missing-permissions (severity: medium)

The workflow file `.github/workflows/test.yml` has no top-level `permissions:` key and none of its three jobs (`build`, `test`, `test-strip-prefix`) define a `permissions:` block. This means the workflow runs with the default GitHub token permissions (which can be broad, including write access to contents and packages). This is especially risky because the workflow is triggered on `pull_request` events, where untrusted code from forks could be run. A minimal `permissions: {}` or specific scopes should be declared.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions

**Notes:**

Fixed all unpinned action references by replacing mutable tags with full commit SHAs (actions/checkout@v2 → @0717577d45739eb3c851188b29f50ed6c0b2194e, actions/github-script@v3 → @ffc2c79a5b2490bd33e0a41c1de74b877714d736). Added top-level `permissions: contents: read` block to test.yml to address the missing-permissions finding. The release-tag.yml already had a permissions block (contents: write, needed for creating releases and pushing branches) so it was left unchanged.

