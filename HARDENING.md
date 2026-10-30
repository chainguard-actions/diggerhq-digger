<!-- markdownlint-disable -->

# Hardening Report: diggerhq--digger/v0.6.155

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **diggerhq--digger/v0.6.155** was hardened automatically. 31 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the step 'digger run' puts an expression straight into the shell command: `run: echo digger run ${{ inputs.id }}`.

Locations:

- `action.yml:279`

### script-injection (severity: high)

Rule (a): the step 'Validate Input Configuration for Google' puts expressions straight into the run script: `if [[ -z ${{ toJSON(inputs.google-auth-credentials) }} && -z "${{ inputs.google-workload-identity-provider }}" ]]` and `"${{ inputs.google-service-account }}"`.

Locations:

- `action.yml:308`

### script-injection (severity: high)

Rule (a): the step 'retrieve cache dir' puts an expression straight into the script, unquoted: `CACHE_DIR=${{ inputs.terraform-cache-dir == '' && format('{0}/cache', github.workspace) || inputs.terraform-cache-dir }}`.

Locations:

- `action.yml:380`

### script-injection (severity: high)

Rule (a): the step 'restore-s3-cache' puts expressions straight into the script: `BUCKET="${{ inputs.cache-dependencies-s3-bucket }}"`, `REGION="${{ inputs.cache-dependencies-s3-region }}"`, `PREFIX="${{ inputs.cache-dependencies-s3-bucket-prefix }}"` and `SCRIPT_PATH="${{ github.action_path }}/scripts/s3-cache-download.bash"`.

Locations:

- `action.yml:397`

### script-injection (severity: high)

Rule (a): the step 'Setup Checkov' puts an expression straight into the script, unquoted: `pip3 install -U checkov==${{ inputs.checkov-version }}`.

Locations:

- `action.yml:500`

### script-injection (severity: high)

Rule (a): the step 'Copy Digger CLI go.sum for cache key' puts expressions straight into the script: `if [[ "${{ inputs.local-dev-mode }}" == "true" ]]` and `elif [[ ${{ inputs.ee }} == "true" ]]` (the second is unquoted).

Locations:

- `action.yml:521`

### script-injection (severity: high)

Rule (a): the step 'build and run digger' puts expressions straight into the run script: `if [[ ${{ inputs.ee }} == "true" ]]` and `if [[ ${{ inputs.fips }} == "true" ]]`.

Locations:

- `action.yml:576`

### script-injection (severity: high)

Rule (a): the step 'download, install, and run digger' puts expressions straight into the run script: `echo "Runner OS: ${{ runner.os }}, Arch: ${{ runner.arch }}, ..."`, `if [[ ${{ inputs.ee }} == "true" ]]`, `if [[ ${{ inputs.fips }} == "true" ]]` and `echo "2. Binary for ${{ runner.os }}-${{ runner.arch }} might not be available"`.

Locations:

- `action.yml:642`

### script-injection (severity: high)

Rule (a): the step 'run digger in local dev mode' puts an expression straight into the script: `RAW="${{ inputs.local-dev-cli-path }}"`. Rule (b): the value built from that input is then run unquoted as `$BIN`.

Locations:

- `action.yml:732`
- `action.yml:754`

### script-injection (severity: high)

Rule (a): the step 'save-s3-cache' puts expressions straight into the script: `BUCKET="${{ inputs.cache-dependencies-s3-bucket }}"`, `REGION="${{ inputs.cache-dependencies-s3-region }}"`, `PREFIX="${{ inputs.cache-dependencies-s3-bucket-prefix }}"` and `SCRIPT_PATH="${{ github.action_path }}/scripts/s3-cache-upload.bash"`.

Locations:

- `action.yml:760`

### github-env-injection (severity: high)

The step 'retrieve cache dir' writes `CACHE_DIR`, which comes from `inputs.terraform-cache-dir`, to $GITHUB_ENV without the `printf '%s' ... | tr -d '\n\r'` sanitization: `echo "TF_PLUGIN_CACHE_DIR=$CACHE_DIR" >> $GITHUB_ENV`, plus the same for TG_PROVIDER_CACHE_DIR and TERRAGRUNT_PROVIDER_CACHE_DIR. A newline in the input lets an attacker inject any environment variable.

Locations:

- `action.yml:383`

### github-env-injection (severity: high)

The step 'Setup tfenv' writes `$TFENV_DIR/bin` to $GITHUB_PATH without sanitization. TFENV_DIR is built from the inherited process env var `HOME` (`TFENV_DIR="${HOME:-/opt}/.tfenv"`; `echo "$TFENV_DIR/bin" >> "$GITHUB_PATH"`). This check treats env vars that the script reads but does not set as caller-controlled.

Locations:

- `action.yml:434`

### github-env-injection (severity: high)

The step 'Setup Terragrunt' writes `$BIN_DIR` to $GITHUB_PATH without sanitization. BIN_DIR is built from the inherited process env var `RUNNER_TEMP` (`BIN_DIR="$RUNNER_TEMP/terragrunt-bin"`; `echo "$BIN_DIR" >> "$GITHUB_PATH"`). This check treats env vars that the script reads but does not set as caller-controlled.

Locations:

- `action.yml:475`

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

Moved all ${{ }} expressions out of run: scripts into step env: blocks (inputs.id, Google validation inputs, terraform-cache-dir expression, S3 cache bucket/region/prefix, checkov-version, local-dev-mode, ee, fips, local-dev-cli-path). Replaced github.action_path with $GITHUB_ACTION_PATH and runner.os/arch with $RUNNER_OS/$RUNNER_ARCH. Quoted "$BIN" in local dev mode. Sanitized values written to GITHUB_ENV (CACHE_DIR) and GITHUB_PATH (tfenv bin, terragrunt BIN_DIR) with printf '%s' | tr -d '\n\r'. The Google validation now checks a plain env var with -z instead of toJSON, which gives the same result. Note: running edits in parallel corrupted the file, so I rewrote it in full from the original content with the fixes applied.

