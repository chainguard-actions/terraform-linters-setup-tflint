<!-- markdownlint-disable -->

# Hardening Report: terraform-linters--setup-tflint/v6.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **terraform-linters--setup-tflint/v6.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a `run:` shell command string. In the `integration-versions` job's "Validate" step, `version='${{ matrix.tflint_version }}'` embeds the matrix value directly into the shell script before the shell ever sees it, allowing an attacker who controls matrix values to inject arbitrary shell commands. The `matrix.*` context is workflow-controllable and must be passed via an `env:` variable and then double-quoted in the shell.

Locations:

- `.github/workflows/test.yml:88`

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a `run:` shell command string. In the `integration-wrapper` job's "Verify Exit Code Output" step, `if [[ ${{ steps.tflint.outputs.exitcode }} -ne 0 ]]; then` embeds a step output directly into the shell script. The `steps.*.outputs.*` context is workflow-controllable and must be passed via an `env:` variable and then double-quoted in the shell.

Locations:

- `.github/workflows/test.yml:129`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in hardened/action/.github/workflows/test.yml:
1. Line 88 (integration-versions job, 'Validate' step): Moved `${{ matrix.tflint_version }}` out of the run: shell string into an `env:` block as `TFLINT_VERSION`, then referenced it as `"${TFLINT_VERSION:1}"` in the shell.
2. Line 129 (integration-wrapper job, 'Verify Exit Code Output' step): Moved `${{ steps.tflint.outputs.exitcode }}` out of the run: shell string into an `env:` block as `TFLINT_EXITCODE`, then referenced it as `"$TFLINT_EXITCODE"` in the shell.

