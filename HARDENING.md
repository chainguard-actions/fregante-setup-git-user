<!-- markdownlint-disable -->

# Hardening Report: fregante--setup-git-user/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fregante--setup-git-user/v1.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` steps in action-release.yml directly interpolate the user-controlled expression `${{ github.event.inputs.Version }}` into shell commands. This allows an attacker who can trigger the workflow_dispatch event to inject arbitrary shell commands. Offending lines:
  - `run: git tag ${{ github.event.inputs.Version }}`
  - `MAJOR=$(echo ${{ github.event.inputs.Version }} | sed 's/\..*//')`
Fix: move the value into an `env:` variable and double-quote it in the shell script.

Locations:

- `.github/workflows/action-release.yml:20`
- `.github/workflows/action-release.yml:27`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags instead of full 40-character commit SHAs, making the workflows vulnerable to supply-chain attacks if those tags are moved:
  - `actions/checkout@v2` (action-release.yml and test.yml)
  - `fregante/release-with-changelog@v3` (action-release.yml)
Fix: pin each reference to a specific commit SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v2`.

Locations:

- `.github/workflows/action-release.yml:19`
- `.github/workflows/action-release.yml:22`
- `.github/workflows/test.yml:10`
- `.github/workflows/test.yml:17`

### missing-permissions (severity: medium)

Neither `.github/workflows/action-release.yml` nor `.github/workflows/test.yml` declares a top-level `permissions:` block, and no job in either file has its own `permissions:` block. Without explicit permissions, workflows run with the default (often broad) token permissions. Fix: add a top-level `permissions: {}` and grant only the specific scopes each job requires (e.g. `contents: write` for the Release job).

Locations:

- `.github/workflows/action-release.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings in .github/workflows/action-release.yml and .github/workflows/test.yml:

1. script-injection: Moved both `${{ github.event.inputs.Version }}` interpolations in action-release.yml into `env:` blocks as `VERSION`, then referenced `"$VERSION"` (double-quoted) in the shell scripts.

2. unpinned-uses: Pinned all mutable tag references to full commit SHAs:
   - actions/checkout@v2 → @0717577d45739eb3c851188b29f50ed6c0b2194e # v2 (both files)
   - fregante/release-with-changelog@v3 → @779fd5e658f82e5b11b1c0a352a6838d3bd8f67f # v3

3. missing-permissions: Added top-level `permissions: {}` to both workflow files (deny-by-default), with job-level `permissions: contents: write` only for jobs that require it (Release job and Test job).

