<!-- markdownlint-disable -->

# Hardening Report: git-for-windows--get-azure-pipelines-artifact/v0.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **git-for-windows--get-azure-pipelines-artifact/v0.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files reference external actions using mutable version tags (@v2, @v3) instead of full 40-character commit SHA hashes. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit.

Failing references in .github/workflows/release-tag.yml:
- uses: actions/checkout@v2
- uses: actions/github-script@v3 (appears twice)

Failing references in .github/workflows/test.yml:
- uses: actions/checkout@v2 (appears three times)

Locations:

- `.github/workflows/release-tag.yml:13`
- `.github/workflows/release-tag.yml:16`
- `.github/workflows/release-tag.yml:60`
- `.github/workflows/test.yml:14`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:26`

### missing-permissions (severity: medium)

Neither .github/workflows/release-tag.yml nor .github/workflows/test.yml defines a top-level `permissions:` key, and no individual job within either file defines job-level `permissions:`. Without explicit permissions, GitHub Actions grants the default token permissions (which can include write access to repository contents, pull requests, etc.), violating the principle of least privilege.

Locations:

- `.github/workflows/release-tag.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions

**Notes:**

Fixed all 6 unpinned action references by replacing mutable version tags with full 40-character commit SHAs (actions/checkout@v2 → @0717577d45739eb3c851188b29f50ed6c0b2194e, actions/github-script@v3 → @ffc2c79a5b2490bd33e0a41c1de74b877714d736). Added top-level permissions blocks to both workflow files: release-tag.yml gets 'contents: write' (required for creating GitHub releases and pushing to branches), and test.yml gets 'contents: read' (minimal access for checkout and testing operations).

