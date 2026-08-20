<!-- markdownlint-disable -->

# Hardening Report: terraform-linters--setup-tflint/v6.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **terraform-linters--setup-tflint/v6.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is directly interpolated inside a `run:` shell command string in the `integration-versions` job. The line `version='${{ matrix.tflint_version }}'` injects the matrix value directly into the shell script before the shell ever sees it. An attacker who can influence the matrix value (e.g. via a workflow_dispatch or repository fork) could inject arbitrary shell commands. Fix: move the value into an `env:` variable and reference it as a quoted shell variable, e.g. `env: TFLINT_VERSION: ${{ matrix.tflint_version }}` then `version="$TFLINT_VERSION"`.

Locations:

- `.github/workflows/test.yml:103`

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is directly interpolated inside a `run:` shell command string in the `integration-wrapper` job. The line `if [[ ${{ steps.tflint.outputs.exitcode }} -ne 0 ]]; then` injects the step output value directly into the shell script. Step outputs can be influenced by the action being tested and could contain shell metacharacters. Fix: move the value into an `env:` variable and reference it as a quoted shell variable, e.g. `env: EXIT_CODE: ${{ steps.tflint.outputs.exitcode }}` then `if [[ "$EXIT_CODE" -ne 0 ]]; then`.

Locations:

- `.github/workflows/test.yml:131`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection findings in .github/workflows/test.yml:
1. Line 103 (integration-versions job): Moved `${{ matrix.tflint_version }}` out of the run: shell string into an `env:` block as `TFLINT_VERSION`, then referenced it as `"$TFLINT_VERSION"` in the shell script.
2. Line 131 (integration-wrapper job): Moved `${{ steps.tflint.outputs.exitcode }}` out of the run: shell string into an `env:` block as `EXIT_CODE`, then referenced it as `"$EXIT_CODE"` in the shell script.

