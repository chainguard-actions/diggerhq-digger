<!-- markdownlint-disable -->

# Hardening Report: diggerhq--digger/v0.6.152

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **diggerhq--digger/v0.6.152** was hardened automatically. 29 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. Step 'digger run ${{github.event.inputs.id}}': the run: block contains `echo digger run ${{ inputs.id }}` — inputs.id is interpolated directly into the shell command string before the shell sees it, enabling command injection.

Locations:

- `action.yml:206`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. Step 'Validate Input Configuration for Google': the run: block uses `${{ toJSON(inputs.google-auth-credentials) }}`, `${{ inputs.google-workload-identity-provider }}`, and `${{ inputs.google-service-account }}` directly inside shell conditional expressions, enabling command injection via attacker-controlled inputs.

Locations:

- `action.yml:237`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. Step 'retrieve cache dir': the run: block assigns `CACHE_DIR=${{ inputs.terraform-cache-dir == '' && format('{0}/cache', github.workspace) || inputs.terraform-cache-dir }}` — both inputs.terraform-cache-dir and github.workspace are interpolated directly into the shell, enabling command injection.

Locations:

- `action.yml:330`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. Step 'restore-s3-cache': the run: block assigns `BUCKET="${{ inputs.cache-dependencies-s3-bucket }}"`, `REGION="${{ inputs.cache-dependencies-s3-region }}"`, `PREFIX="${{ inputs.cache-dependencies-s3-bucket-prefix }}"`, and `SCRIPT_PATH="${{ github.action_path }}/..."` — all are interpolated directly into the shell before execution, enabling command injection.

Locations:

- `action.yml:347`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. Step 'Setup Checkov': the run: block contains `pip3 install -U checkov==${{ inputs.checkov-version }}` — inputs.checkov-version is interpolated directly into the shell command, enabling command injection via a malicious version string.

Locations:

- `action.yml:450`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. Step 'Copy Digger CLI go.sum for cache key': the run: block uses `${{ inputs.local-dev-mode }}` and `${{ inputs.ee }}` directly inside shell conditional expressions (`if [[ "${{ inputs.local-dev-mode }}" == "true" ]]` and `elif [[ ${{ inputs.ee }} == "true" ]]`), enabling command injection.

Locations:

- `action.yml:460`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. Step 'build and run digger': the run: block uses `if [[ ${{ inputs.ee }} == "true" ]]` and `if [[ ${{ inputs.fips }} == "true" ]]` — inputs.ee and inputs.fips are interpolated directly into shell conditionals, enabling command injection.

Locations:

- `action.yml:472`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. Step 'download, install, and run digger': the run: block uses `${{ runner.os }}`, `${{ runner.arch }}`, `${{ inputs.ee }}`, and `${{ inputs.fips }}` directly inside shell commands (e.g., `echo "Runner OS: ${{ runner.os }}, Arch: ${{ runner.arch }}"` and `if [[ ${{ inputs.ee }} == "true" ]]`), enabling injection.

Locations:

- `action.yml:625`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. Step 'run digger in local dev mode': the run: block assigns `RAW="${{ inputs.local-dev-cli-path }}"` — inputs.local-dev-cli-path is interpolated directly into the shell before the validation logic runs, enabling command injection before the guard is reached.

Locations:

- `action.yml:730`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. Step 'save-s3-cache': the run: block assigns `BUCKET="${{ inputs.cache-dependencies-s3-bucket }}"`, `REGION="${{ inputs.cache-dependencies-s3-region }}"`, `PREFIX="${{ inputs.cache-dependencies-s3-bucket-prefix }}"`, and `SCRIPT_PATH="${{ github.action_path }}/..."` — all are interpolated directly into the shell, enabling command injection.

Locations:

- `action.yml:755`

### github-env-injection (severity: high)

Step 'retrieve cache dir': the run: block writes `CACHE_DIR` (derived from `${{ inputs.terraform-cache-dir }}` and `${{ github.workspace }}`, both untrusted inputs) to $GITHUB_ENV via `echo "TF_PLUGIN_CACHE_DIR=$CACHE_DIR" >> $GITHUB_ENV` (and similarly for TG_PROVIDER_CACHE_DIR and TERRAGRUNT_PROVIDER_CACHE_DIR) without applying the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker can inject newlines to set arbitrary environment variables.

Locations:

- `action.yml:333`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.id }}" appears directly in run: block of step "digger run ${{github.event.inputs.id}}"; move to env: map

Locations:

- `action.yml:280`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.google-workload-identity-provider }}" appears directly in run: block of step "Validate Input Configuration for Google"; move to env: map

Locations:

- `action.yml:308`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.google-workload-identity-provider }}" appears directly in run: block of step "Validate Input Configuration for Google"; move to env: map

Locations:

- `action.yml:310`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.google-service-account }}" appears directly in run: block of step "Validate Input Configuration for Google"; move to env: map

Locations:

- `action.yml:310`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-dependencies-s3-bucket }}" appears directly in run: block of step "restore-s3-cache"; move to env: map

Locations:

- `action.yml:400`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-dependencies-s3-region }}" appears directly in run: block of step "restore-s3-cache"; move to env: map

Locations:

- `action.yml:401`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-dependencies-s3-bucket-prefix }}" appears directly in run: block of step "restore-s3-cache"; move to env: map

Locations:

- `action.yml:402`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.checkov-version }}" appears directly in run: block of step "Setup Checkov"; move to env: map

Locations:

- `action.yml:500`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.local-dev-mode }}" appears directly in run: block of step "Copy Digger CLI go.sum for cache key"; move to env: map

Locations:

- `action.yml:521`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ee }}" appears directly in run: block of step "Copy Digger CLI go.sum for cache key"; move to env: map

Locations:

- `action.yml:523`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ee }}" appears directly in run: block of step "build and run digger"; move to env: map

Locations:

- `action.yml:584`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fips }}" appears directly in run: block of step "build and run digger"; move to env: map

Locations:

- `action.yml:589`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ee }}" appears directly in run: block of step "download, install, and run digger"; move to env: map

Locations:

- `action.yml:644`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fips }}" appears directly in run: block of step "download, install, and run digger"; move to env: map

Locations:

- `action.yml:645`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.local-dev-cli-path }}" appears directly in run: block of step "run digger in local dev mode"; move to env: map

Locations:

- `action.yml:731`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-dependencies-s3-bucket }}" appears directly in run: block of step "save-s3-cache"; move to env: map

Locations:

- `action.yml:762`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-dependencies-s3-region }}" appears directly in run: block of step "save-s3-cache"; move to env: map

Locations:

- `action.yml:763`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-dependencies-s3-bucket-prefix }}" appears directly in run: block of step "save-s3-cache"; move to env: map

Locations:

- `action.yml:764`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Fixed all 29 findings across 10 steps in action.yml:

1. **'digger run' step**: Moved `${{ inputs.id }}` to env block as `INPUT_ID`.

2. **'Validate Input Configuration for Google' step**: Moved `${{ toJSON(inputs.google-auth-credentials) }}`, `${{ inputs.google-workload-identity-provider }}`, and `${{ inputs.google-service-account }}` to env block as `GOOGLE_AUTH_CREDENTIALS`, `GOOGLE_WORKLOAD_IDENTITY_PROVIDER`, `GOOGLE_SERVICE_ACCOUNT`.

3. **'retrieve cache dir' step** (script-injection + github-env-injection): Moved `${{ inputs.terraform-cache-dir }}` and `${{ github.workspace }}` to env block; replaced the inline expression with conditional shell logic; added `printf '%s' "$CACHE_DIR" | tr -d '\n\r'` sanitization before writing to GITHUB_ENV.

4. **'restore-s3-cache' step**: Moved `${{ inputs.cache-dependencies-s3-bucket }}`, `${{ inputs.cache-dependencies-s3-region }}`, `${{ inputs.cache-dependencies-s3-bucket-prefix }}`, and `${{ github.action_path }}` to env block as `INPUT_S3_BUCKET`, `INPUT_S3_REGION`, `INPUT_S3_PREFIX`, `ACTION_PATH`.

5. **'Setup Checkov' step**: Moved `${{ inputs.checkov-version }}` to env block as `CHECKOV_VERSION`.

6. **'Copy Digger CLI go.sum for cache key' step**: Moved `${{ inputs.local-dev-mode }}` and `${{ inputs.ee }}` to env block as `INPUT_LOCAL_DEV_MODE` and `INPUT_EE`.

7. **'build and run digger' step**: Added `INPUT_EE: ${{ inputs.ee }}` and `INPUT_FIPS: ${{ inputs.fips }}` to the existing env block; replaced inline expressions in run block with env var references.

8. **'download, install, and run digger' step**: Added `INPUT_EE`, `INPUT_FIPS`, `RUNNER_OS_VAL`, `RUNNER_ARCH_VAL` to env block; replaced all inline `${{ runner.os }}`, `${{ runner.arch }}`, `${{ inputs.ee }}`, `${{ inputs.fips }}` with env var references.

9. **'run digger in local dev mode' step**: Added `INPUT_LOCAL_DEV_CLI_PATH: ${{ inputs.local-dev-cli-path }}` to env block; replaced `RAW="${{ inputs.local-dev-cli-path }}"` with `RAW="$INPUT_LOCAL_DEV_CLI_PATH"`.

10. **'save-s3-cache' step**: Moved `${{ inputs.cache-dependencies-s3-bucket }}`, `${{ inputs.cache-dependencies-s3-region }}`, `${{ inputs.cache-dependencies-s3-bucket-prefix }}`, and `${{ github.action_path }}` to env block as `INPUT_S3_BUCKET`, `INPUT_S3_REGION`, `INPUT_S3_PREFIX`, `ACTION_PATH`.

