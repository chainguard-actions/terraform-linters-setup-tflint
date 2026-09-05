<!-- markdownlint-disable -->

# Hardening Report: terraform-linters--setup-tflint/v6.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **terraform-linters--setup-tflint/v6.3.1** was hardened automatically. 2 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in test.yml directly interpolate `${{ }}` expressions inside shell scripts, violating rule (a). This allows template substitution to inject arbitrary shell metacharacters before the shell parses the command.

Offending lines:
- `version='${{ matrix.tflint_version }}'` (Validate step, integration-versions job)
- `if [[ "${{ matrix.tflint_version }}" == "latest" ]];` (Verify version output step)
- `version='${{ matrix.tflint_version }}'` (same step)
- `if [[ "${{ steps.setup-tflint.outputs.tflint-version }}" != "$expected" ]];` (same step)
- `echo "... got '${{ steps.setup-tflint.outputs.tflint-version }}'"` (same step)
- `if [[ "${{ steps.setup-tflint.outputs.tflint-version }}" != "0.52.0" ]];` (Verify version output, integration-version-file job)
- `if [[ ${{ steps.tflint.outputs.exitcode }} -ne 0 ]];` (Verify Exit Code Output, integration-wrapper job)
- `if [[ -z "${{ steps.setup-tflint.outputs.tflint-version }}" ]];` (Verify Version Output, integration-wrapper job)
- `echo "TFLint version captured: ${{ steps.setup-tflint.outputs.tflint-version }}"` (same step)

Fix: Move values into `env:` variables and reference them as quoted shell variables (e.g., `"$TFLINT_VERSION"`) instead of using `${{ }}` directly in the script body.

Locations:

- `.github/workflows/test.yml:88`
- `.github/workflows/test.yml:93`
- `.github/workflows/test.yml:96`
- `.github/workflows/test.yml:98`
- `.github/workflows/test.yml:99`
- `.github/workflows/test.yml:137`
- `.github/workflows/test.yml:138`
- `.github/workflows/test.yml:172`
- `.github/workflows/test.yml:179`
- `.github/workflows/test.yml:181`

### script-injection (severity: high)

A `run:` block in release.yml directly interpolates a `${{ steps.version.outputs.tag }}` expression inside a shell command, violating rule (a). The step output value is substituted by the GitHub Actions template engine before the shell parses the command, allowing a crafted tag value to inject shell metacharacters.

Offending line:
`run: npm version "${{ steps.version.outputs.tag }}" --no-git-tag-version --allow-same-version`

Fix: Move the value into an `env:` variable (e.g., `TAG: ${{ steps.version.outputs.tag }}`) and reference it as `"$TAG"` inside the run script.

Locations:

- `.github/workflows/release.yml:68`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection findings by moving ${{ }} expressions from run: shell script bodies into env: blocks:

**release.yml (line 68):**
- Added `env: TAG: ${{ steps.version.outputs.tag }}` to the 'Bump version' step and changed `npm version "${{ steps.version.outputs.tag }}"` to `npm version "$TAG"`

**test.yml - integration-versions job:**
- 'Validate' step (non-latest): Added `env: TFLINT_VERSION: ${{ matrix.tflint_version }}` and changed `version='${{ matrix.tflint_version }}'` to `version="$TFLINT_VERSION"`
- 'Verify version output' step: Added `TFLINT_VERSION` and `TFLINT_VERSION_OUTPUT` env vars; replaced all 4 occurrences of `${{ matrix.tflint_version }}` and `${{ steps.setup-tflint.outputs.tflint-version }}` with `$TFLINT_VERSION` and `$TFLINT_VERSION_OUTPUT`

**test.yml - integration-version-file job:**
- 'Verify version output' step: Added `env: TFLINT_VERSION_OUTPUT: ${{ steps.setup-tflint.outputs.tflint-version }}` and replaced both occurrences of `${{ steps.setup-tflint.outputs.tflint-version }}` with `$TFLINT_VERSION_OUTPUT`

**test.yml - integration-wrapper job:**
- 'Verify Exit Code Output' step: Added `env: TFLINT_EXITCODE: ${{ steps.tflint.outputs.exitcode }}` and replaced `${{ steps.tflint.outputs.exitcode }}` with `$TFLINT_EXITCODE`
- 'Verify Version Output' step: Added `env: TFLINT_VERSION_OUTPUT: ${{ steps.setup-tflint.outputs.tflint-version }}` and replaced both occurrences of `${{ steps.setup-tflint.outputs.tflint-version }}` with `$TFLINT_VERSION_OUTPUT`

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted shell expansion in the `integration-versions` job's "Validate" step in `.github/workflows/test.yml`. Changed `grep ${version:1}` to `grep "${version:1}"` to prevent shell metacharacter interpretation (glob patterns, word splitting, etc.) from the workflow-controllable `matrix.tflint_version` value.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in .github/workflows/release.yml at the 'Determine version' step. Added explicit sanitization using `printf '%s' ... | tr -d '\n\r'` for the `tag`, `major`, and `minor` variables before writing them to `$GITHUB_OUTPUT`. These values are derived from the user-controlled `inputs.version` workflow_dispatch input. The sanitization is applied after the regex validation but immediately before the write to `$GITHUB_OUTPUT`, satisfying the requirement for explicit sanitization at every write point.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansion in .github/workflows/test.yml at line 213. Changed `if [[ $TFLINT_EXITCODE -ne 0 ]]` to `if [[ "$TFLINT_EXITCODE" -ne 0 ]]` to properly quote the TFLINT_EXITCODE variable that is sourced from the workflow-controllable `steps.tflint.outputs.exitcode` value.

