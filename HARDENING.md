<!-- markdownlint-disable -->

# Hardening Report: fregante--setup-git-user/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fregante--setup-git-user/v2.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The workflow directly interpolates the user-controlled expression `${{ github.event.inputs.Version }}` inside two `run:` shell command strings. An attacker with workflow_dispatch access can inject arbitrary shell commands. Offending lines:
  1. `run: gh release create ${{ github.event.inputs.Version }} --generate-notes`
  2. `MAJOR=$(echo ${{ github.event.inputs.Version }} | sed 's/\..*//')`
Fix: move the value into an `env:` variable and double-quote it in the shell script.

Locations:

- `.github/workflows/action-release.yml:20`
- `.github/workflows/action-release.yml:24`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the GITHUB_TOKEN is granted its default (often broad) permissions. Add a top-level `permissions: {}` block and grant only the minimal scopes required (e.g., `contents: write` for creating releases and pushing tags).

Locations:

- `.github/workflows/action-release.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the GITHUB_TOKEN is granted its default (often broad) permissions. Add a top-level `permissions: {}` block and grant only the minimal scopes required (e.g., `contents: write` for pushing to branches).

Locations:

- `.github/workflows/test.yml:1`

### unpinned-uses (severity: high)

The following `uses:` references are pinned to a mutable tag (`@v3`) rather than an immutable 40-character commit SHA. A compromised or force-pushed tag could silently substitute malicious code.
  - `uses: actions/checkout@v3`
Fix: pin to a full SHA, e.g. `uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`.

Locations:

- `.github/workflows/action-release.yml:19`
- `.github/workflows/test.yml:8`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions, unpinned-uses

**Notes:**

Fixed all four findings across two workflow files:
1. action-release.yml — script-injection: moved `${{ github.event.inputs.Version }}` into `env: VERSION:` on both affected steps; the shell now references `"$VERSION"` (double-quoted) instead of the raw expression.
2. action-release.yml — missing-permissions: added `permissions: contents: write` at the top level (required for `gh release create` and `git push --force`).
3. test.yml — missing-permissions: added `permissions: contents: write` at the top level (required for `git push` to branches).
4. Both files — unpinned-uses: pinned `actions/checkout@v3` to full SHA `a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3`.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted $MAJOR variable in the 'Update major tag' step of .github/workflows/action-release.yml. Changed `git push origin HEAD:refs/tags/$MAJOR --force` to `git push origin HEAD:"refs/tags/$MAJOR" --force`. This prevents shell metacharacter injection from the user-controlled `Version` workflow_dispatch input, which is derived into $MAJOR via sed.

