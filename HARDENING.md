<!-- markdownlint-disable -->

# Hardening Report: fregante--setup-git-user/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fregante--setup-git-user/v2.0.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 1e985c9..e0432c3 100644
--- a/action.yml
+++ b/action.yml
@@ -4,10 +4,26 @@ author: Federico Brigante
 branding:
   icon: chevron-up
   color: yellow
-
 runs:
   using: composite
   steps:
     - shell: bash
       run: |
         git config --global user.name "GitHub Actions"

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** prior-fix

**Notes:**

Re-applied the visible part of the prior patch (removed a blank line before runs:). The diff shown was cut off and its +16 added lines were not included, so they could not be reproduced. action.yml contains only two static 'git config --global' bash steps: no ${{ }} expressions, no uses:, no downloads, no credentials, no GITHUB_ENV/OUTPUT writes. No security change is needed.

