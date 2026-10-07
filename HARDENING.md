<!-- markdownlint-disable -->

# Hardening Report: diggerhq--digger/v0.6.153

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **diggerhq--digger/v0.6.153** was hardened automatically. 29 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolated inside a run: shell command. The first step uses `run: echo digger run ${{ inputs.id }}` — the expression is substituted by the Actions runner before the shell sees it, enabling script injection via the `inputs.id` value.

Locations:

- `action.yml:236`

### script-injection (severity: high)

Rule (a): The 'Validate Input Configuration for Google' run: block directly interpolates `${{ toJSON(inputs.google-auth-credentials) }}`, `${{ inputs.google-workload-identity-provider }}`, and `${{ inputs.google-service-account }}` inside shell `if [[ ... ]]` conditionals. These expressions are substituted before the shell executes, allowing injection via attacker-controlled input values.

Locations:

- `action.yml:261`

### script-injection (severity: high)

Rule (a): The 'retrieve cache dir' run: block directly interpolates `${{ inputs.terraform-cache-dir == '' && format('{0}/cache', github.workspace) || inputs.terraform-cache-dir }}` as a shell assignment (`CACHE_DIR=...`). The `inputs.terraform-cache-dir` and `github.workspace` values are substituted before the shell executes, enabling injection.

Locations:

- `action.yml:352`

### github-env-injection (severity: high)

The 'retrieve cache dir' run: block assigns `CACHE_DIR` from a direct `${{ inputs.terraform-cache-dir ... }}` expression (which includes the caller-controlled `inputs.terraform-cache-dir`) and then writes it to $GITHUB_ENV via `echo "TF_PLUGIN_CACHE_DIR=$CACHE_DIR" >> $GITHUB_ENV` (and similarly for TG_PROVIDER_CACHE_DIR and TERRAGRUNT_PROVIDER_CACHE_DIR) without the required `printf '%s' ... | tr -d '\n\r'` sanitization. A newline in the input value can inject arbitrary environment variables.

Locations:

- `action.yml:355`

### script-injection (severity: high)

Rule (a): The 'restore-s3-cache' run: block directly interpolates `${{ inputs.cache-dependencies-s3-bucket }}`, `${{ inputs.cache-dependencies-s3-region }}`, `${{ inputs.cache-dependencies-s3-bucket-prefix }}`, and `${{ github.action_path }}` inside shell variable assignments. These expressions are substituted before the shell executes.

Locations:

- `action.yml:372`

### script-injection (severity: high)

Rule (a): The 'Setup Checkov' run: block directly interpolates `${{ inputs.checkov-version }}` inside a pip install command: `pip3 install -U checkov==${{ inputs.checkov-version }}`. An attacker-controlled checkov-version value is substituted directly into the shell command before execution.

Locations:

- `action.yml:481`

### script-injection (severity: high)

Rule (a): The 'Copy Digger CLI go.sum for cache key' run: block directly interpolates `${{ inputs.local-dev-mode }}` and `${{ inputs.ee }}` inside shell `if [[ ... ]]` conditionals. These expressions are substituted before the shell executes.

Locations:

- `action.yml:502`

### script-injection (severity: high)

Rule (a): The 'build and run digger' run: block directly interpolates `${{ inputs.ee }}` and `${{ inputs.fips }}` inside shell `if [[ ... ]]` conditionals. These expressions are substituted before the shell executes, enabling injection via attacker-controlled input values.

Locations:

- `action.yml:562`

### script-injection (severity: high)

Rule (a): The 'download, install, and run digger' run: block directly interpolates `${{ runner.os }}`, `${{ runner.arch }}`, `${{ inputs.ee }}`, and `${{ inputs.fips }}` inside shell commands (echo and if-conditionals). Any ${{ ... }} expression inside a run: block is a script-injection risk regardless of context.

Locations:

- `action.yml:643`

### script-injection (severity: high)

Rule (a): The 'run digger in local dev mode' run: block directly interpolates `${{ inputs.local-dev-cli-path }}` as `RAW="${{ inputs.local-dev-cli-path }}"`. Although the script attempts to validate the value afterward, the expression is substituted by the Actions runner before the shell executes, so shell metacharacters in the input value are processed before any validation can occur.

Locations:

- `action.yml:762`

### script-injection (severity: high)

Rule (a): The 'save-s3-cache' run: block directly interpolates `${{ inputs.cache-dependencies-s3-bucket }}`, `${{ inputs.cache-dependencies-s3-region }}`, `${{ inputs.cache-dependencies-s3-bucket-prefix }}`, and `${{ github.action_path }}` inside shell variable assignments. These expressions are substituted before the shell executes.

Locations:

- `action.yml:822`

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

Fixed all 29 findings across 11 steps in action.yml:

1. 'digger run' step: moved ${{ inputs.id }} to env block as INPUT_ID
2. 'Validate Input Configuration for Google': moved google-auth-credentials, google-workload-identity-provider, google-service-account to env block
3. 'retrieve cache dir': moved terraform-cache-dir and github.workspace to env block; added printf/tr sanitization before writing to GITHUB_ENV
4. 'restore-s3-cache': moved cache-dependencies-s3-bucket, cache-dependencies-s3-region, cache-dependencies-s3-bucket-prefix, github.action_path to env block
5. 'Setup Checkov': moved checkov-version to env block as CHECKOV_VERSION
6. 'Copy Digger CLI go.sum for cache key': moved local-dev-mode and ee to env block
7. 'build and run digger': added INPUT_EE and INPUT_FIPS to existing env block; updated run block to use env vars
8. 'download, install, and run digger': added INPUT_EE, INPUT_FIPS, RUNNER_OS_VAL, RUNNER_ARCH_VAL to env block; updated run block
9. 'run digger in local dev mode': added INPUT_LOCAL_DEV_CLI_PATH to env block; updated run block
10. 'save-s3-cache': moved cache-dependencies-s3-bucket, cache-dependencies-s3-region, cache-dependencies-s3-bucket-prefix, github.action_path to env block

