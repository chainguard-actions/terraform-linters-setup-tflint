<!-- markdownlint-disable -->

# Hardening Report: terraform-linters--setup-tflint/v6.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **terraform-linters--setup-tflint/v6.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in .github/workflows/test.yml directly interpolate GitHub Actions expressions inside shell command strings, enabling script injection. Affected locations:

- `integration-versions` job, "Validate" step (line ~84): `version='${{ matrix.tflint_version }}'`
- `integration-versions` job, "Verify version output" step (lines ~89-94): `if [[ "${{ matrix.tflint_version }}" == "latest" ]];`, `version='${{ matrix.tflint_version }}'`, `if [[ "${{ steps.setup-tflint.outputs.tflint-version }}" != "$expected" ]];`, and `echo "...'${{ steps.setup-tflint.outputs.tflint-version }}'"` 
- `integration-version-file` job, "Verify version output" step (lines ~132-133): `if [[ "${{ steps.setup-tflint.outputs.tflint-version }}" != "0.52.0" ]];` and `echo "...'${{ steps.setup-tflint.outputs.tflint-version }}'"` 
- `integration-wrapper` job, "Verify Exit Code Output" step (line ~164): `if [[ ${{ steps.tflint.outputs.exitcode }} -ne 0 ]];` (unquoted)
- `integration-wrapper` job, "Verify Version Output" step (lines ~170-173): `if [[ -z "${{ steps.setup-tflint.outputs.tflint-version }}" ]];` and `echo "TFLint version captured: ${{ steps.setup-tflint.outputs.tflint-version }}"`

All of these should route values through `env:` variables and reference them as `"$VAR"` in the shell, never interpolating `${{ }}` directly inside a `run:` block.

Locations:

- `.github/workflows/test.yml:84`
- `.github/workflows/test.yml:89`
- `.github/workflows/test.yml:91`
- `.github/workflows/test.yml:93`
- `.github/workflows/test.yml:94`
- `.github/workflows/test.yml:132`
- `.github/workflows/test.yml:133`
- `.github/workflows/test.yml:164`
- `.github/workflows/test.yml:170`
- `.github/workflows/test.yml:173`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection issues in hardened/action/.github/workflows/test.yml by moving GitHub Actions expressions out of run: shell blocks and into env: blocks:

1. integration-versions / 'Validate' step: moved `${{ matrix.tflint_version }}` to env var TFLINT_VERSION
2. integration-versions / 'Verify version output' step: moved `${{ matrix.tflint_version }}` to TFLINT_VERSION and `${{ steps.setup-tflint.outputs.tflint-version }}` to TFLINT_VERSION_OUTPUT; preserved existing GH_TOKEN env var
3. integration-version-file / 'Verify version output' step: moved `${{ steps.setup-tflint.outputs.tflint-version }}` to TFLINT_VERSION_OUTPUT
4. integration-wrapper / 'Verify Exit Code Output' step: moved `${{ steps.tflint.outputs.exitcode }}` to TFLINT_EXITCODE and added quotes around the variable in the comparison
5. integration-wrapper / 'Verify Version Output' step: moved `${{ steps.setup-tflint.outputs.tflint-version }}` to TFLINT_VERSION_OUTPUT

All shell scripts now reference plain environment variables instead of directly interpolating ${{ }} expressions.

