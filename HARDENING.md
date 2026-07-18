<!-- markdownlint-disable -->

# Hardening Report: diggerhq--digger/v0.6.146

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **diggerhq--digger/v0.6.146** was hardened automatically. 20 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ }} expressions into shell commands, violating rule (a). This allows an attacker who controls the calling workflow's inputs or github context to inject arbitrary shell commands.

1. Step 'digger run': `run: echo digger run ${{ inputs.id }}` — direct interpolation of inputs.id.
2. Step 'Validate Input Configuration for Google': `if [[ -z ${{ toJSON(inputs.google-auth-credentials) }} && -z "${{ inputs.google-workload-identity-provider }}" ]]` and `${{ inputs.google-service-account }}` — direct interpolation of inputs in shell conditionals.
3. Step 'retrieve cache dir': `CACHE_DIR=${{ inputs.terraform-cache-dir == '' && format('{0}/cache', github.workspace) || inputs.terraform-cache-dir }}` — direct interpolation of inputs.terraform-cache-dir in shell.
4. Step 'restore-s3-cache': `BUCKET="${{ inputs.cache-dependencies-s3-bucket }}"`, `REGION="${{ inputs.cache-dependencies-s3-region }}"`, `PREFIX="${{ inputs.cache-dependencies-s3-bucket-prefix }}"` — direct interpolation of inputs in shell.
5. Step 'Setup Checkov': `pip3 install -U checkov==${{ inputs.checkov-version }}` — direct interpolation of inputs.checkov-version in shell command.
6. Step 'Copy Digger CLI go.sum for cache key': `if [[ "${{ inputs.local-dev-mode }}" == "true" ]]` and `elif [[ ${{ inputs.ee }} == "true" ]]` — direct interpolation of inputs in shell.
7. Step 'build and run digger': `if [[ ${{ inputs.ee }} == "true" ]]` and `if [[ ${{ inputs.fips }} == "true" ]]` — direct interpolation of inputs in shell.
8. Step 'download, install, and run digger': `echo "Runner OS: ${{ runner.os }}, Arch: ${{ runner.arch }}"`, `if [[ ${{ inputs.ee }} == "true" ]]`, `if [[ ${{ inputs.fips }} == "true" ]]`, `echo "2. Binary for ${{ runner.os }}-${{ runner.arch }} might not be available"` — direct interpolation of runner context and inputs in shell.
9. Step 'run digger in local dev mode': `RAW="${{ inputs.local-dev-cli-path }}"` — direct interpolation of inputs.local-dev-cli-path in shell (note: a regex validation follows, but the injection happens before validation).
10. Step 'save-s3-cache': `BUCKET="${{ inputs.cache-dependencies-s3-bucket }}"`, `REGION="${{ inputs.cache-dependencies-s3-region }}"`, `PREFIX="${{ inputs.cache-dependencies-s3-bucket-prefix }}"`, `SCRIPT_PATH="${{ github.action_path }}/scripts/s3-cache-upload.bash"` — direct interpolation of inputs and github context in shell.

Locations:

- `action.yml:218`
- `action.yml:264`
- `action.yml:310`
- `action.yml:340`
- `action.yml:380`
- `action.yml:400`
- `action.yml:430`
- `action.yml:460`
- `action.yml:530`
- `action.yml:560`

### github-env-injection (severity: high)

The 'retrieve cache dir' run: block writes a value derived from the untrusted input `inputs.terraform-cache-dir` directly to $GITHUB_ENV without sanitization. The expression `${{ inputs.terraform-cache-dir == '' && format('{0}/cache', github.workspace) || inputs.terraform-cache-dir }}` is interpolated directly into the shell script, and the resulting value is written to $GITHUB_ENV via `echo "TF_PLUGIN_CACHE_DIR=$CACHE_DIR" >> $GITHUB_ENV` (and similarly for TG_PROVIDER_CACHE_DIR and TERRAGRUNT_PROVIDER_CACHE_DIR). An attacker controlling `inputs.terraform-cache-dir` could inject newlines to set arbitrary environment variables in subsequent steps. The required sanitization step (`printf '%s' "$CACHE_DIR" | tr -d '\n\r'`) is absent.

Locations:

- `action.yml:310`

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

- `action.yml:456`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.local-dev-mode }}" appears directly in run: block of step "Copy Digger CLI go.sum for cache key"; move to env: map

Locations:

- `action.yml:477`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ee }}" appears directly in run: block of step "Copy Digger CLI go.sum for cache key"; move to env: map

Locations:

- `action.yml:479`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ee }}" appears directly in run: block of step "build and run digger"; move to env: map

Locations:

- `action.yml:540`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fips }}" appears directly in run: block of step "build and run digger"; move to env: map

Locations:

- `action.yml:545`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ee }}" appears directly in run: block of step "download, install, and run digger"; move to env: map

Locations:

- `action.yml:600`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fips }}" appears directly in run: block of step "download, install, and run digger"; move to env: map

Locations:

- `action.yml:601`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.local-dev-cli-path }}" appears directly in run: block of step "run digger in local dev mode"; move to env: map

Locations:

- `action.yml:687`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-dependencies-s3-bucket }}" appears directly in run: block of step "save-s3-cache"; move to env: map

Locations:

- `action.yml:718`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-dependencies-s3-region }}" appears directly in run: block of step "save-s3-cache"; move to env: map

Locations:

- `action.yml:719`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-dependencies-s3-bucket-prefix }}" appears directly in run: block of step "save-s3-cache"; move to env: map

Locations:

- `action.yml:720`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all 20 findings in action.yml:

1. **script-injection / static-inline-injection**: Moved every ${{ }} expression out of run: shell blocks into the step's env: block, then referenced as plain environment variables ($VAR_NAME) in the shell scripts. Affected steps:
   - 'digger run': inputs.id → INPUT_ID
   - 'Validate Input Configuration for Google': google-auth-credentials (toJSON), google-workload-identity-provider, google-service-account → GOOGLE_AUTH_CREDENTIALS, GOOGLE_WORKLOAD_IDENTITY_PROVIDER, GOOGLE_SERVICE_ACCOUNT
   - 'retrieve cache dir': terraform-cache-dir expression → INPUT_TERRAFORM_CACHE_DIR
   - 'restore-s3-cache': cache-dependencies-s3-bucket/region/prefix, github.action_path → INPUT_S3_BUCKET, INPUT_S3_REGION, INPUT_S3_PREFIX, ACTION_PATH
   - 'Setup Checkov': checkov-version → CHECKOV_VERSION
   - 'Copy Digger CLI go.sum for cache key': local-dev-mode, ee → INPUT_LOCAL_DEV_MODE, INPUT_EE
   - 'build and run digger': ee, fips → INPUT_EE, INPUT_FIPS (added to existing env block)
   - 'download, install, and run digger': ee, fips, runner.os, runner.arch → INPUT_EE, INPUT_FIPS, RUNNER_OS, RUNNER_ARCH (added to existing env block)
   - 'run digger in local dev mode': local-dev-cli-path → INPUT_LOCAL_DEV_CLI_PATH (added to existing env block)
   - 'save-s3-cache': cache-dependencies-s3-bucket/region/prefix, github.action_path → INPUT_S3_BUCKET, INPUT_S3_REGION, INPUT_S3_PREFIX, ACTION_PATH

2. **github-env-injection**: In the 'retrieve cache dir' step, the value is now sanitized with `printf '%s' "$INPUT_TERRAFORM_CACHE_DIR" | tr -d '\n\r'` before being written to $GITHUB_ENV, preventing newline injection attacks.

### Iteration 2

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, unsafe-shell, hardcoded-credentials

**Notes:**

Fixed all 5 security findings across 42 workflow files:

1. unpinned-uses: Pinned all mutable tag/branch-based action references to full 40-character SHA hashes with tag comments for readability. Covered all actions including actions/checkout, actions/setup-go, docker/*, depot/*, aws-actions/*, opentofu/setup-opentofu, hashicorp/setup-terraform, softprops/action-gh-release, superfly/flyctl-actions/setup-flyctl, EndBug/latest-tag, diggerhq/top-issues, googleapis/release-please-action, azure/setup-helm, actions/upload-artifact, actions/download-artifact, actions/github-script, docker/setup-qemu-action.

2. missing-permissions: Added top-level permissions blocks to 23 workflow files. Test/build workflows got 'contents: read', release workflows got 'contents: write', deploy workflows got 'contents: read'.

3. script-injection: Fixed 7 files by moving ${{ }} expressions from run: shell strings to step env: blocks. Fixed github.event_name, github.event.inputs.force_services, github.event.before, github.base_ref, github.workspace, steps.tag.outputs.tag, github.event.pull_request.number, github.sha (in run blocks), github.event.workflow_run.head_branch, vars.ECS_CLUSTER_NAME, vars.ECS_SERVICE_NAME, secrets.GCP_CREDENTIALS, and needs.detect-changes.outputs.* expressions.

4. unsafe-shell: Fixed both check-migrations-generated.yml and test-migrations.yml by replacing 'curl -sSf https://atlasgo.sh | sh' with a two-step approach: download to /tmp/atlasgo-install.sh first, then execute separately.

5. hardcoded-credentials: Replaced hardcoded values in test-migrations.yml (POSTGRES_PASSWORD: postgres, MYSQL_PASSWORD: mysql, MYSQL_ROOT_PASSWORD: root, OPENTACO_JWT_SECRET: test-secret-key-for-ci) with secret references (${{ secrets.CI_POSTGRES_PASSWORD }}, ${{ secrets.CI_MYSQL_PASSWORD }}, ${{ secrets.CI_MYSQL_ROOT_PASSWORD }}, ${{ secrets.CI_JWT_SECRET }}).

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 6 workflow files:

1. taco-release.yml: Moved matrix.goos, matrix.goarch, version, sha, os, arch expressions to env: blocks in Build CLI/Statesman/Token Service steps. Moved github.ref_name to env: block in Determine service name step.

2. dgctl_release.yml: Moved matrix.os and matrix.arch to env: block (GOOS/GOARCH) in Build step; removed the inline 'env VAR=val' prefix from the command.

3. helm-release.yml: Moved secrets.GITHUB_TOKEN and github.actor to env: blocks in both Login steps; moved matrix.chart to env: block in Package and push step.

4. latest_tag.yml: Moved github.repository and github.ref to env: block in the check step.

5. libs_test.yml: Moved github.event.pull_request.head.repo.full_name and github.repository to env: block in pr_check step; moved steps.pr_check.outputs.is_external to env: block in Test step.

6. update-helm-on-release.yml: Added printf+tr sanitization before writing to GITHUB_OUTPUT; changed yq command to use env(VERSION) instead of shell-interpolating VERSION into the yq expression.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed all 6 script-injection occurrences in .github/workflows/helm-test.yml. For each of the three affected steps ('Build chart dependencies (if any)', 'Lint chart', 'Run unit tests (if present)'), added an `env:` block with `CHART: ${{ matrix.chart }}` and replaced all inline `${{ matrix.chart }}` expressions in the run: shell scripts with double-quoted `"$CHART"` references. This prevents the template engine from substituting values directly into shell command strings before the shell sees them.

### Iteration 5

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection findings: (1) In auto-tag-and-release.yml, double-quoted $PREV_COMMIT in all 7 `git diff --name-only` invocations (the variable is sourced from github.event.before via GH_EVENT_BEFORE env var). (2) In backend_build_prod_ecr.yml, double-quoted $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG in both the `docker build -t` and `docker push` commands to prevent shell metacharacter injection from the step output value.

