<!-- markdownlint-disable -->

# Hardening Report: git-for-windows--get-azure-pipelines-artifact/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **git-for-windows--get-azure-pipelines-artifact/v0.3.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'commit & push' step in npm-run-package.yml directly interpolates `${{github.actor}}` inside a `run:` shell block. The `github.actor` value is attacker-influenced (e.g. via a crafted username) and flows through YAML template substitution before the shell sees it, enabling command injection. Offending lines: `git config user.name "${{github.actor}}"` and `git config user.email "${{github.actor}}@users.noreply.github.com"`. These should be moved to an `env:` block and the env var double-quoted in the shell script.

Locations:

- `.github/workflows/npm-run-package.yml:33`
- `.github/workflows/npm-run-package.yml:34`

### unpinned-uses (severity: high)

All `uses:` references across all three workflow files use mutable tag-based refs instead of immutable 40-character SHA commit digests, making the workflows vulnerable to supply-chain attacks if the referenced tags are moved or compromised. Failing references include: actions/checkout@v4, actions/github-script@v7, actions/upload-artifact@v2.

Locations:

- `.github/workflows/npm-run-package.yml:18`
- `.github/workflows/release-tag.yml:11`
- `.github/workflows/release-tag.yml:15`
- `.github/workflows/release-tag.yml:57`
- `.github/workflows/test.yml:14`
- `.github/workflows/test.yml:24`
- `.github/workflows/test.yml:31`
- `.github/workflows/test.yml:40`
- `.github/workflows/test.yml:44`

### missing-permissions (severity: medium)

npm-run-package.yml and test.yml have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions. A top-level or per-job `permissions:` block with minimal required scopes should be added to both files.

Locations:

- `.github/workflows/npm-run-package.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across the three workflow files:
1. script-injection (npm-run-package.yml): Moved github.actor into an env: block (ACTOR) and referenced it as $ACTOR in the shell script, preventing YAML template injection.
2. unpinned-uses: Pinned all mutable tag refs to full 40-char SHAs with tag comments: actions/checkout@v4 → SHA 11d5960a..., actions/github-script@v7 → SHA f28e40c7..., actions/upload-artifact@v2 → SHA 82c141cc...
3. missing-permissions: Added top-level permissions blocks to npm-run-package.yml (contents: write, needed for git push) and test.yml (contents: read, minimal for checkout/test). release-tag.yml already had permissions: contents: write.

