<!-- markdownlint-disable -->

# Hardening Report: terraform-linters--setup-tflint/v6.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **terraform-linters--setup-tflint/v6.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Validate' step in the integration-versions job directly interpolates `${{ matrix.tflint_version }}` inside a `run:` shell command: `version='${{ matrix.tflint_version }}'`. The matrix value is substituted by the YAML template engine before the shell ever sees it, allowing an attacker who controls the matrix value to inject arbitrary shell commands.

Locations:

- `.github/workflows/test.yml:104`

### script-injection (severity: high)

Sub-rule (a): The 'Verify version output' step in the integration-versions job directly interpolates `${{ matrix.tflint_version }}` and `${{ steps.setup-tflint.outputs.tflint-version }}` inside a `run:` shell script. Offending lines include: `if [[ "${{ matrix.tflint_version }}" == "latest" ]]; then`, `version='${{ matrix.tflint_version }}'`, `if [[ "${{ steps.setup-tflint.outputs.tflint-version }}" != "$expected" ]]; then`, and the echo line. Both matrix context and step outputs are workflow-controllable and must not be interpolated directly into shell.

Locations:

- `.github/workflows/test.yml:111`

### script-injection (severity: high)

Sub-rule (a): The 'Verify Exit Code Output' step in the integration-wrapper job directly interpolates `${{ steps.tflint.outputs.exitcode }}` inside a `run:` shell command: `if [[ ${{ steps.tflint.outputs.exitcode }} -ne 0 ]]; then`. Step outputs are workflow-controllable and must not be interpolated directly into shell; they should be passed via an env: variable and then double-quoted in the script.

Locations:

- `.github/workflows/test.yml:143`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three script injection vulnerabilities in hardened/action/.github/workflows/test.yml:
1. 'Validate' step (line ~104): Moved `${{ matrix.tflint_version }}` into env var `TFLINT_VERSION` and referenced it as `"$TFLINT_VERSION"` in the shell script.
2. 'Verify version output' step (line ~111): Moved `${{ matrix.tflint_version }}` into `TFLINT_VERSION` and `${{ steps.setup-tflint.outputs.tflint-version }}` into `TFLINT_VERSION_OUTPUT` env vars; updated all shell references to use these env vars.
3. 'Verify Exit Code Output' step (line ~143): Moved `${{ steps.tflint.outputs.exitcode }}` into `TFLINT_EXITCODE` env var and double-quoted it in the arithmetic comparison `[[ "$TFLINT_EXITCODE" -ne 0 ]]`.

