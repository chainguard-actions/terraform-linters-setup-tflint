<!-- markdownlint-disable -->

# Hardening Report: terraform-linters--setup-tflint/v6.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **terraform-linters--setup-tflint/v6.2.2** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in .github/workflows/test.yml directly interpolate ${{ ... }} expressions inside shell commands (sub-rule a). This means GitHub Actions performs YAML template substitution before the shell ever sees the value, allowing shell metacharacters to be injected.

1. (line ~104) `version='${{ matrix.tflint_version }}'` in `run: |-` block — matrix value interpolated directly into shell.
2. (line ~111) `if [[ "${{ matrix.tflint_version }}" == "latest" ]];` in `run: |` block.
3. (line ~114) `version='${{ matrix.tflint_version }}'` in same run block.
4. (line ~117) `if [[ "${{ steps.setup-tflint.outputs.tflint-version }}" != "$expected" ]];` — step output interpolated directly.
5. (line ~118) `echo "Expected version output '$expected', got '${{ steps.setup-tflint.outputs.tflint-version }}'"` — step output interpolated directly.
6. (line ~143) `if [[ ${{ steps.tflint.outputs.exitcode }} -ne 0 ]];` — step output interpolated directly and unquoted.
7. (line ~149) `if [[ -z "${{ steps.setup-tflint.outputs.tflint-version }}" ]];` — step output interpolated directly.
8. (line ~152) `echo "TFLint version captured: ${{ steps.setup-tflint.outputs.tflint-version }}"` — step output interpolated directly.

Fix: move the values into env: variables and reference them as quoted shell variables (e.g., "$TFLINT_VERSION") inside the run: block.

Locations:

- `.github/workflows/test.yml:104`
- `.github/workflows/test.yml:111`
- `.github/workflows/test.yml:114`
- `.github/workflows/test.yml:117`
- `.github/workflows/test.yml:118`
- `.github/workflows/test.yml:143`
- `.github/workflows/test.yml:149`
- `.github/workflows/test.yml:152`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 8 script injection instances in .github/workflows/test.yml by moving ${{ ... }} expressions into env: blocks and referencing them as quoted shell variables:
1. 'Validate' step in integration-versions: added env: TFLINT_VERSION for ${{ matrix.tflint_version }}
2. 'Verify version output' step: added env: TFLINT_VERSION and TFLINT_VERSION_OUTPUT for ${{ matrix.tflint_version }} and ${{ steps.setup-tflint.outputs.tflint-version }}
3. 'Verify Exit Code Output' step in integration-wrapper: added env: TFLINT_EXITCODE for ${{ steps.tflint.outputs.exitcode }} and added quotes around the variable in the comparison
4. 'Verify Version Output' step in integration-wrapper: added env: TFLINT_VERSION_OUTPUT for ${{ steps.setup-tflint.outputs.tflint-version }}

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansion in `.github/workflows/test.yml` line 93: changed `grep ${version:1}` to `grep "${version:1}"`. This prevents shell metacharacters in the matrix `tflint_version` value from being interpreted by the shell when used as a grep argument.

