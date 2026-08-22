<!-- markdownlint-disable -->

# Hardening Report: terraform-linters--setup-tflint/v6.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **terraform-linters--setup-tflint/v6.2.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple `run:` blocks in the `integration-versions` job directly interpolate `${{ matrix.tflint_version }}` and `${{ steps.setup-tflint.outputs.tflint-version }}` expressions inside shell commands. These values flow through YAML template substitution before the shell parses them, allowing an attacker who controls matrix values or step outputs to inject arbitrary shell commands.

Offending lines in the 'Validate' step:
  `version='${{ matrix.tflint_version }}'`

Offending lines in the 'Verify version output' step:
  `if [[ "${{ matrix.tflint_version }}" == "latest" ]]; then`
  `version='${{ matrix.tflint_version }}'`
  `if [[ "${{ steps.setup-tflint.outputs.tflint-version }}" != "$expected" ]]; then`
  `echo "Expected version output '$expected', got '${{ steps.setup-tflint.outputs.tflint-version }}'"` 

Fix: move values into `env:` variables and reference them as quoted shell variables (e.g., `"$TFLINT_VERSION"`).

Locations:

- `.github/workflows/test.yml:94`
- `.github/workflows/test.yml:99`
- `.github/workflows/test.yml:101`
- `.github/workflows/test.yml:103`
- `.github/workflows/test.yml:105`

### script-injection (severity: high)

Rule (a): Two `run:` blocks in the `integration-wrapper` job directly interpolate `${{ steps.tflint.outputs.exitcode }}` (unquoted) and `${{ steps.setup-tflint.outputs.tflint-version }}` inside shell commands. Step outputs are workflow-controllable and flow through YAML template substitution before the shell parses them.

Offending lines in 'Verify Exit Code Output' step:
  `if [[ ${{ steps.tflint.outputs.exitcode }} -ne 0 ]]; then`  (also unquoted — rule (b) violation)

Offending lines in 'Verify Version Output' step:
  `if [[ -z "${{ steps.setup-tflint.outputs.tflint-version }}" ]]; then`
  `echo "TFLint version captured: ${{ steps.setup-tflint.outputs.tflint-version }}"`

Fix: move values into `env:` variables and reference them as quoted shell variables.

Locations:

- `.github/workflows/test.yml:120`
- `.github/workflows/test.yml:126`
- `.github/workflows/test.yml:128`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection findings in .github/workflows/test.yml:

1. integration-versions job, 'Validate' step (tflint_version != 'latest'): Moved `${{ matrix.tflint_version }}` into env var `TFLINT_VERSION` and referenced it as `"$TFLINT_VERSION"` in the shell script.

2. integration-versions job, 'Verify version output' step: Moved `${{ matrix.tflint_version }}` into `TFLINT_VERSION` and `${{ steps.setup-tflint.outputs.tflint-version }}` into `TFLINT_VERSION_OUTPUT` env vars; updated all shell references accordingly. Kept existing `GH_TOKEN` env var in the same block.

3. integration-wrapper job, 'Verify Exit Code Output' step: Moved `${{ steps.tflint.outputs.exitcode }}` into env var `TFLINT_EXITCODE` and referenced it as `"$TFLINT_EXITCODE"` (also added quotes to fix the unquoted rule (b) violation).

4. integration-wrapper job, 'Verify Version Output' step: Moved `${{ steps.setup-tflint.outputs.tflint-version }}` into env var `TFLINT_VERSION_OUTPUT` and referenced it as `"$TFLINT_VERSION_OUTPUT"` throughout the shell script.

