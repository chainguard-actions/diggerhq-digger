<!-- markdownlint-disable -->

# Hardening Report: diggerhq--digger/v0.6.154

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **diggerhq--digger/v0.6.154** was hardened automatically. 20 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions (sub-rule a), allowing script injection. Affected steps and offending lines:

1. Step 'digger run': `run: echo digger run ${{ inputs.id }}` — inputs.id is interpolated directly into the shell command.

2. Step 'Validate Input Configuration for Google': `if [[ -z ${{ toJSON(inputs.google-auth-credentials) }} && -z "${{ inputs.google-workload-identity-provider }}" ]]` and `elif [[ ! -z "${{ inputs.google-workload-identity-provider }}" && -z "${{ inputs.google-service-account }}" ]]` — multiple inputs interpolated directly into shell conditionals.

3. Step 'retrieve cache dir': `CACHE_DIR=${{ inputs.terraform-cache-dir == '' && format('{0}/cache', github.workspace) || inputs.terraform-cache-dir }}` — inputs.terraform-cache-dir and github.workspace interpolated directly into the shell.

4. Step 'restore-s3-cache': `BUCKET="${{ inputs.cache-dependencies-s3-bucket }}"`, `REGION="${{ inputs.cache-dependencies-s3-region }}"`, `PREFIX="${{ inputs.cache-dependencies-s3-bucket-prefix }}"`, `SCRIPT_PATH="${{ github.action_path }}"` — multiple inputs and github context interpolated directly.

5. Step 'Setup Checkov': `pip3 install -U checkov==${{ inputs.checkov-version }}` — inputs.checkov-version interpolated directly into a pip install command.

6. Step 'Copy Digger CLI go.sum': `if [[ "${{ inputs.local-dev-mode }}" == "true" ]]` and `elif [[ ${{ inputs.ee }} == "true" ]]` — inputs interpolated directly into shell conditionals.

7. Step 'build and run digger': `if [[ ${{ inputs.ee }} == "true" ]]` and `if [[ ${{ inputs.fips }} == "true" ]]` — inputs interpolated directly into shell conditionals.

8. Step 'download, install, and run digger': `echo "Runner OS: ${{ runner.os }}, Arch: ${{ runner.arch }}"`, `if [[ ${{ inputs.ee }} == "true" ]]`, `if [[ ${{ inputs.fips }} == "true" ]]` — runner context and inputs interpolated directly into shell.

9. Step 'run digger in local dev mode': `RAW="${{ inputs.local-dev-cli-path }}"` — inputs.local-dev-cli-path interpolated directly into the shell.

10. Step 'save-s3-cache': `BUCKET="${{ inputs.cache-dependencies-s3-bucket }}"`, `REGION="${{ inputs.cache-dependencies-s3-region }}"`, `PREFIX="${{ inputs.cache-dependencies-s3-bucket-prefix }}"`, `SCRIPT_PATH="${{ github.action_path }}"` — multiple inputs and github context interpolated directly.

Locations:

- `action.yml:280`
- `action.yml:226`
- `action.yml:293`
- `action.yml:309`
- `action.yml:386`
- `action.yml:403`
- `action.yml:431`
- `action.yml:491`
- `action.yml:571`
- `action.yml:641`

### github-env-injection (severity: high)

The 'retrieve cache dir' step writes `$CACHE_DIR` to `$GITHUB_ENV` (and also `$TG_PROVIDER_CACHE_DIR` and `$TERRAGRUNT_PROVIDER_CACHE_DIR`) without sanitization. The value of `CACHE_DIR` is set directly from the `${{ inputs.terraform-cache-dir }}` expression interpolated inline in the `run:` block — not via an `env:` variable. An attacker-controlled `inputs.terraform-cache-dir` value containing newlines could inject arbitrary environment variables into `$GITHUB_ENV`. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent before the write.

Offending code:
```
CACHE_DIR=${{ inputs.terraform-cache-dir == '' &&
  format('{0}/cache', github.workspace) ||
  inputs.terraform-cache-dir }}
echo "TF_PLUGIN_CACHE_DIR=$CACHE_DIR" >> $GITHUB_ENV
echo "TG_PROVIDER_CACHE_DIR=$CACHE_DIR" >> $GITHUB_ENV
echo "TERRAGRUNT_PROVIDER_CACHE_DIR=$CACHE_DIR" >> $GITHUB_ENV
```

Locations:

- `action.yml:293`

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

Fixed all 20 findings across 10 steps in action.yml:

1. **Step 'digger run'**: Moved `${{ inputs.id }}` to env block as `INPUT_ID`, referenced as `"$INPUT_ID"` in run.

2. **Step 'Validate Input Configuration for Google'**: Moved `${{ toJSON(inputs.google-auth-credentials) }}`, `${{ inputs.google-workload-identity-provider }}`, and `${{ inputs.google-service-account }}` to env block as `GOOGLE_AUTH_CREDENTIALS`, `GOOGLE_WORKLOAD_IDENTITY_PROVIDER`, `GOOGLE_SERVICE_ACCOUNT`.

3. **Step 'retrieve cache dir'** (script-injection + github-env-injection): Moved `${{ inputs.terraform-cache-dir }}` and `${{ github.workspace }}` to env block. Added sanitization with `printf '%s' "$RAW_CACHE_DIR" | tr -d '\n\r'` before writing to `$GITHUB_ENV`.

4. **Step 'restore-s3-cache'**: Moved `${{ inputs.cache-dependencies-s3-bucket }}`, `${{ inputs.cache-dependencies-s3-region }}`, `${{ inputs.cache-dependencies-s3-bucket-prefix }}`, and `${{ github.action_path }}` to env block.

5. **Step 'Setup Checkov'**: Moved `${{ inputs.checkov-version }}` to env block as `CHECKOV_VERSION`, used as `"checkov==${CHECKOV_VERSION}"` in pip install.

6. **Step 'Copy Digger CLI go.sum for cache key'**: Moved `${{ inputs.local-dev-mode }}` and `${{ inputs.ee }}` to env block as `INPUT_LOCAL_DEV_MODE` and `INPUT_EE`.

7. **Step 'build and run digger'**: Added `INPUT_EE: ${{ inputs.ee }}` and `INPUT_FIPS: ${{ inputs.fips }}` to existing env block, replaced inline expressions in run: block.

8. **Step 'download, install, and run digger'**: Added `INPUT_EE`, `INPUT_FIPS`, `RUNNER_OS_NAME`, `RUNNER_ARCH_NAME` to existing env block, replaced all inline `${{ runner.os }}`, `${{ runner.arch }}`, `${{ inputs.ee }}`, `${{ inputs.fips }}` expressions in run: block.

9. **Step 'run digger in local dev mode'**: Added `INPUT_LOCAL_DEV_CLI_PATH: ${{ inputs.local-dev-cli-path }}` to existing env block, replaced `RAW="${{ inputs.local-dev-cli-path }}"` with `RAW="$INPUT_LOCAL_DEV_CLI_PATH"`.

10. **Step 'save-s3-cache'**: Moved `${{ inputs.cache-dependencies-s3-bucket }}`, `${{ inputs.cache-dependencies-s3-region }}`, `${{ inputs.cache-dependencies-s3-bucket-prefix }}`, and `${{ github.action_path }}` to env block.

