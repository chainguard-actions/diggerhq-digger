<!-- markdownlint-disable -->

# Hardening Report: diggerhq--digger/v0.6.151

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **diggerhq--digger/v0.6.151** was hardened automatically. 20 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ }} expressions into shell commands, violating rule (a). This allows an attacker to inject arbitrary shell commands via controlled inputs.

1. Step 'digger run': `run: echo digger run ${{ inputs.id }}` — direct interpolation of inputs.id.

2. Step 'retrieve cache dir': `CACHE_DIR=${{ inputs.terraform-cache-dir == '' && format('{0}/cache', github.workspace) || inputs.terraform-cache-dir }}` — direct interpolation of inputs.terraform-cache-dir and github.workspace in the run: block.

3. Step 'restore-s3-cache': `BUCKET="${{ inputs.cache-dependencies-s3-bucket }}"`, `REGION="${{ inputs.cache-dependencies-s3-region }}"`, `PREFIX="${{ inputs.cache-dependencies-s3-bucket-prefix }}"`, `SCRIPT_PATH="${{ github.action_path }}/scripts/s3-cache-download.bash"` — direct interpolation of inputs.* in run: block.

4. Step 'Setup Checkov': `pip3 install -U checkov==${{ inputs.checkov-version }}` — direct interpolation of inputs.checkov-version in run: block.

5. Step 'Copy Digger CLI go.sum for cache key': `if [[ "${{ inputs.local-dev-mode }}" == "true" ]]` and `elif [[ ${{ inputs.ee }} == "true" ]]` — direct interpolation of inputs.* in run: block.

6. Step 'build and run digger': `if [[ ${{ inputs.ee }} == "true" ]]` and `if [[ ${{ inputs.fips }} == "true" ]]` — direct interpolation of inputs.* in run: block.

7. Step 'download, install, and run digger': `echo "Runner OS: ${{ runner.os }}, Arch: ${{ runner.arch }}, Digger Version: ${DIGGER_VERSION}"`, `if [[ ${{ inputs.ee }} == "true" ]]`, `if [[ ${{ inputs.fips }} == "true" ]]`, and `echo "2. Binary for ${{ runner.os }}-${{ runner.arch }} might not be available"` — direct interpolation of runner.* and inputs.* in run: block.

8. Step 'run digger in local dev mode': `RAW="${{ inputs.local-dev-cli-path }}"` — direct interpolation of inputs.local-dev-cli-path in run: block.

9. Step 'save-s3-cache': `BUCKET="${{ inputs.cache-dependencies-s3-bucket }}"`, `REGION="${{ inputs.cache-dependencies-s3-region }}"`, `PREFIX="${{ inputs.cache-dependencies-s3-bucket-prefix }}"`, `SCRIPT_PATH="${{ github.action_path }}/scripts/s3-cache-upload.bash"` — direct interpolation of inputs.* in run: block.

Locations:

- `action.yml:233`
- `action.yml:331`
- `action.yml:351`
- `action.yml:432`
- `action.yml:461`
- `action.yml:533`
- `action.yml:601`
- `action.yml:701`
- `action.yml:762`

### github-env-injection (severity: high)

The 'retrieve cache dir' step writes $CACHE_DIR to $GITHUB_ENV without sanitization. The value of CACHE_DIR is derived directly from `${{ inputs.terraform-cache-dir }}` (and `${{ github.workspace }}`), which are interpolated inline in the run: block. An attacker controlling the `terraform-cache-dir` input can inject newlines into $GITHUB_ENV to set arbitrary environment variables for subsequent steps. The lines:
  `echo "TF_PLUGIN_CACHE_DIR=$CACHE_DIR" >> $GITHUB_ENV`
  `echo "TG_PROVIDER_CACHE_DIR=$CACHE_DIR" >> $GITHUB_ENV`
  `echo "TERRAGRUNT_PROVIDER_CACHE_DIR=$CACHE_DIR" >> $GITHUB_ENV`
are not preceded by the required sanitization step (`printf '%s' "$CACHE_DIR" | tr -d '\n\r'`).

Locations:

- `action.yml:331`

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

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all 20 security findings in hardened/action/action.yml:

1. Step 'digger run': Moved inputs.id to env block as INPUT_ID.

2. Step 'Validate Input Configuration for Google': Moved google-auth-credentials, google-workload-identity-provider, and google-service-account to env block.

3. Step 'retrieve cache dir': Moved inputs.terraform-cache-dir and github.workspace to env block, replaced inline ternary expression with if/else logic, and added printf/tr sanitization before writing to GITHUB_ENV to prevent newline injection.

4. Step 'restore-s3-cache': Moved cache-dependencies-s3-bucket, cache-dependencies-s3-region, cache-dependencies-s3-bucket-prefix, and github.action_path to env block.

5. Step 'Setup Checkov': Moved inputs.checkov-version to env block as CHECKOV_VERSION.

6. Step 'Copy Digger CLI go.sum for cache key': Moved inputs.local-dev-mode and inputs.ee to env block.

7. Step 'build and run digger': Added INPUT_EE and INPUT_FIPS to existing env block, updated run block to use env vars.

8. Step 'download, install, and run digger': Added INPUT_EE, INPUT_FIPS, RUNNER_OS_VAL, RUNNER_ARCH_VAL to existing env block, updated run block to use env vars.

9. Step 'run digger in local dev mode': Added INPUT_LOCAL_DEV_CLI_PATH to existing env block, updated run block to use env var.

10. Step 'save-s3-cache': Moved cache-dependencies-s3-bucket, cache-dependencies-s3-region, cache-dependencies-s3-bucket-prefix, and github.action_path to env block.

