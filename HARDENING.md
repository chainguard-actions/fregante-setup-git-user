<!-- markdownlint-disable -->

# Hardening Report: fregante--setup-git-user/v2.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fregante--setup-git-user/v2.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The workflow directly interpolates the untrusted user-supplied input `${{ github.event.inputs.Version }}` inside two `run:` shell command strings. An attacker with workflow_dispatch access can inject arbitrary shell commands. Offending lines:
  Line 21: `run: gh release create ${{ github.event.inputs.Version }} --generate-notes`
  Line 24: `MAJOR=$(echo ${{ github.event.inputs.Version }} | sed 's/\..*//')`
Fix: move the value into an `env:` variable and double-quote every expansion, e.g. `env: { VERSION: "${{ github.event.inputs.Version }}" }` then use `"$VERSION"` in the script.

Locations:

- `.github/workflows/action-release.yml:21`
- `.github/workflows/action-release.yml:24`

### unpinned-uses (severity: high)

Both workflow files reference `actions/checkout@v3` using a mutable version tag instead of a pinned 40-character commit SHA. A compromised or altered tag could introduce malicious code into the workflow. All unpinned references:
  - `.github/workflows/action-release.yml`: `uses: actions/checkout@v3`
  - `.github/workflows/test.yml`: `uses: actions/checkout@v3`
Fix: pin each reference to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`.

Locations:

- `.github/workflows/action-release.yml:19`
- `.github/workflows/test.yml:9`

### missing-permissions (severity: medium)

Neither `.github/workflows/action-release.yml` nor `.github/workflows/test.yml` declares a top-level `permissions:` block, and neither of their jobs declares job-level `permissions:`. Without explicit permissions, workflows run with the repository's default token permissions (often `write-all`), violating the principle of least privilege. Fix: add a top-level `permissions: {}` (or specific minimal scopes such as `contents: write` for the release workflow) to each file.

Locations:

- `.github/workflows/action-release.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across .github/workflows/action-release.yml and .github/workflows/test.yml:

1. script-injection (action-release.yml lines 21, 24): Moved `${{ github.event.inputs.Version }}` into `env: VERSION:` blocks for both affected steps. The `gh release create` step now uses `"$VERSION"` and the `Update major tag` step uses `echo "$VERSION"` in the shell script.

2. unpinned-uses (both files): Pinned `actions/checkout@v3` to its full commit SHA `a37ce9120846195fa4ece8f58b268e6043cb2f26` with a `# v3` comment in both action-release.yml and test.yml.

3. missing-permissions (both files): Added `permissions: contents: write` at the top level of both workflow files. `contents: write` is the minimum needed for creating releases and pushing tags (action-release.yml) and for pushing/deleting branches (test.yml).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansion in `.github/workflows/action-release.yml` line 27. Changed `git push origin HEAD:refs/tags/$MAJOR --force` to `git push origin "HEAD:refs/tags/$MAJOR" --force`. The $MAJOR variable is derived from the user-controlled `github.event.inputs.Version` input, and leaving it unquoted allowed shell metacharacters to potentially break out of the intended command. Wrapping the refspec in double quotes ensures the value is treated as a single argument regardless of its content.

