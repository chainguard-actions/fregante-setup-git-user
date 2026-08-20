<!-- markdownlint-disable -->

# Hardening Report: fregante--setup-git-user/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fregante--setup-git-user/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files reference actions using mutable version tags (@v3) instead of pinned full-length SHA digests. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit. Failing references: `actions/checkout@v3` in action-release.yml and `actions/checkout@v3` in test.yml.

Locations:

- `.github/workflows/action-release.yml:20`
- `.github/workflows/test.yml:9`

### script-injection (severity: high)

Rule (a): `${{ github.event.inputs.Version }}` is interpolated directly inside two `run:` shell command strings in action-release.yml. An attacker with workflow_dispatch access can supply a value containing shell metacharacters to achieve arbitrary command execution. Offending lines: `run: gh release create ${{ github.event.inputs.Version }} --generate-notes` and `MAJOR=$(echo ${{ github.event.inputs.Version }} | sed 's/\..*//')`. Rule (b): The shell variable `$MAJOR` (derived from the untrusted input above) is used unquoted in `git push origin HEAD:refs/tags/$MAJOR --force`, allowing shell metacharacter injection.

Locations:

- `.github/workflows/action-release.yml:21`
- `.github/workflows/action-release.yml:24`

### missing-permissions (severity: medium)

Neither workflow file defines a top-level `permissions:` key, and neither job within them defines a job-level `permissions:` key. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad (e.g. write access to contents, packages, etc.).

Locations:

- `.github/workflows/action-release.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across both workflow files:

1. **unpinned-uses**: Pinned `actions/checkout@v3` to full SHA `a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3` in both action-release.yml and test.yml.

2. **script-injection** (action-release.yml): Moved `${{ github.event.inputs.Version }}` out of both `run:` shell strings into `env:` blocks as `VERSION`. Referenced as `"$VERSION"` (double-quoted) in the shell. Also double-quoted `"$MAJOR"` in the `git push` command to prevent shell metacharacter injection from the derived variable. Used `printf '%s'` for safe string handling.

3. **missing-permissions**: Added `permissions: contents: write` at the top level of both workflow files. `contents: write` is the minimum required — needed for creating releases and pushing tags (action-release.yml) and pushing/deleting branches (test.yml).

