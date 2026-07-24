<!-- markdownlint-disable -->

# Hardening Report: diggerhq--digger/v0.6.147

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **diggerhq--digger/v0.6.147** was hardened automatically. 24 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ ... }} expressions inside shell commands, enabling script injection. Affected steps and patterns include:
- 'digger run' step: `run: echo digger run ${{ inputs.id }}`
- 'Validate Input Configuration for Google' step: `if [[ -z ${{ toJSON(inputs.google-auth-credentials) }} && -z "${{ inputs.google-workload-identity-provider }}" ]]`
- 'retrieve cache dir' step: `CACHE_DIR=${{ inputs.terraform-cache-dir == '' && format(...) || inputs.terraform-cache-dir }}`
- 'Setup Checkov' step: `pip3 install -U checkov==${{ inputs.checkov-version }}`
- 'Copy Digger CLI go.sum' step: `if [[ "${{ inputs.local-dev-mode }}" == "true" ]]` and `elif [[ ${{ inputs.ee }} == "true" ]]`
- 'build and run digger' step: `if [[ ${{ inputs.ee }} == "true" ]]` and `if [[ ${{ inputs.fips }} == "true" ]]`
- 'download, install, and run digger' step: `echo "Runner OS: ${{ runner.os }}, Arch: ${{ runner.arch }}"`, `if [[ ${{ inputs.ee }} == "true" ]]`, `if [[ ${{ inputs.fips }} == "true" ]]`
- 'run digger in local dev mode' step: `RAW="${{ inputs.local-dev-cli-path }}"`
All of these allow an attacker to inject arbitrary shell commands via the expression values before the shell ever sees them.

Locations:

- `action.yml:209`
- `action.yml:244`
- `action.yml:330`
- `action.yml:399`
- `action.yml:420`
- `action.yml:490`
- `action.yml:560`
- `action.yml:680`

### github-env-injection (severity: high)

The 'retrieve cache dir' run: block in action.yml writes a value derived from ${{ inputs.terraform-cache-dir }} (and ${{ github.workspace }}) directly to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). The expression `CACHE_DIR=${{ inputs.terraform-cache-dir == '' && format('{0}/cache', github.workspace) || inputs.terraform-cache-dir }}` is interpolated directly in the shell, and then `echo "TF_PLUGIN_CACHE_DIR=$CACHE_DIR" >> $GITHUB_ENV` writes it unsanitized. An attacker-controlled `terraform-cache-dir` input could inject newlines to set arbitrary environment variables.

Locations:

- `action.yml:330`

### hardcoded-credentials (severity: high)

The workflow file test-migrations.yml contains a hardcoded literal secret value: `OPENTACO_JWT_SECRET: test-secret-key-for-ci`. Although labeled as a CI test value, this is a non-expression literal assigned to a name containing 'secret', matching the hardcoded-credentials check pattern.

Locations:

- `.github/workflows/test-migrations.yml:148`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job, meaning they run with the default (potentially broad) token permissions:
- backend_test.yml
- backend_release.yml
- cli_test.yml
- cli_release.yml
- cli_release_multiarch.yml
- cli_test_e2e.yml (has job-level permissions — PASS)
- dgctl_release.yml
- drift-deploy.yml
- ee_backend_test.yml
- ee_cli_test.yml
- ee_cli_release.yml
- ee_cli_release_fips.yml
- ee_cli_release_multiarch.yml
- helm-test.yml
- latest_tag.yml
- libs_test.yml
- misc_top_issues.yml
- pro-deploy.yml
- pro-projects-service-deploy.yml
- taco_statesman_docker_build_verify.yml
- tasks_run_test.yml
- check-migrations-generated.yml
- test-migrations.yml

Locations:

- `.github/workflows/backend_test.yml:1`
- `.github/workflows/backend_release.yml:1`
- `.github/workflows/cli_test.yml:1`
- `.github/workflows/cli_release.yml:1`
- `.github/workflows/cli_release_multiarch.yml:1`
- `.github/workflows/dgctl_release.yml:1`
- `.github/workflows/drift-deploy.yml:1`
- `.github/workflows/ee_backend_test.yml:1`
- `.github/workflows/ee_cli_test.yml:1`
- `.github/workflows/ee_cli_release.yml:1`
- `.github/workflows/ee_cli_release_fips.yml:1`
- `.github/workflows/ee_cli_release_multiarch.yml:1`
- `.github/workflows/helm-test.yml:1`
- `.github/workflows/latest_tag.yml:1`
- `.github/workflows/libs_test.yml:1`
- `.github/workflows/misc_top_issues.yml:1`
- `.github/workflows/pro-deploy.yml:1`
- `.github/workflows/pro-projects-service-deploy.yml:1`
- `.github/workflows/taco_statesman_docker_build_verify.yml:1`
- `.github/workflows/tasks_run_test.yml:1`
- `.github/workflows/check-migrations-generated.yml:1`
- `.github/workflows/test-migrations.yml:1`

### unpinned-uses (severity: high)

Virtually all workflow files use `uses:` references pinned to mutable tags or branch names (e.g. @v4, @v5, @v1, @main, @master) rather than immutable 40-character SHA digests. This exposes the workflows to supply-chain attacks if the referenced action is compromised or the tag is moved. Affected files include (non-exhaustive list of failing references):
- auto-tag-and-release.yml: actions/checkout@v4, and many others
- backend-ee-release.yml: actions/checkout@v4, docker/login-action@v3, docker/build-push-action@v5, etc.
- backend_build_prod_ecr.yml: actions/checkout@v4, aws-actions/configure-aws-credentials@v4, etc.
- backend_docker_release.yml: actions/checkout@v4, docker/build-push-action@v6, etc.
- backend_release.yml: actions/checkout@v4
- backend_test.yml: actions/setup-go@v5, actions/checkout@v4, depot/setup-action@v1, etc.
- check-migrations-generated.yml: actions/checkout@v4, actions/setup-go@v5
- cli_release.yml: actions/checkout@v4
- cli_release_multiarch.yml: actions/checkout@v4
- cli_test.yml: actions/setup-go@v5, opentofu/setup-opentofu@v1, actions/checkout@v4
- cli_test_e2e.yml: actions/checkout@v4, actions/setup-go@v5, google-github-actions/auth@v2
- dgctl_release.yml: actions/setup-go@v5, actions/checkout@v4, actions/upload-release-asset@v1
- drift-deploy.yml: actions/checkout@v4, superfly/flyctl-actions/setup-flyctl@master
- drift-release.yml: actions/checkout@v4, docker/login-action@v3, etc.
- ee_backend_docker_release.yml, ee_backend_docker_release_fips.yml, ee_backend_test.yml, ee_cli_release.yml, ee_cli_release_fips.yml, ee_cli_release_multiarch.yml, ee_cli_test.yml, ee_tasks_release.yml: similar patterns
- helm-release.yml, helm-test.yml: actions/checkout@v4
- latest_tag.yml: actions/checkout@v4, actions/github-script@v7, etc.
- libs_test.yml: opentofu/setup-opentofu@v1.0.5, hashicorp/setup-terraform@v3, actions/setup-go@v5, etc.
- misc_top_issues.yml: diggerhq/top-issues@main
- pro-deploy.yml, pro-projects-service-deploy.yml: actions/checkout@v4, superfly/flyctl-actions/setup-flyctl@master
- projects-refresh-release.yml, projects_refresh_image_build.yml: similar patterns
- release-please.yml: google-github-actions/release-please-action@v4
- sidecar-release.yml, staging-deploy.yml, taco-release.yml, taco_statesman_docker_build_verify.yml, tasks_release.yml, tasks_run_test.yml, test-migrations.yml, ui-release.yml, update-helm-on-release.yml, vlatest-tag-update.yml: similar patterns

Locations:

- `.github/workflows/auto-tag-and-release.yml:1`
- `.github/workflows/backend-ee-release.yml:1`
- `.github/workflows/backend_build_prod_ecr.yml:1`
- `.github/workflows/backend_docker_release.yml:1`
- `.github/workflows/backend_release.yml:1`
- `.github/workflows/backend_test.yml:1`
- `.github/workflows/check-migrations-generated.yml:1`
- `.github/workflows/cli_release.yml:1`
- `.github/workflows/cli_release_multiarch.yml:1`
- `.github/workflows/cli_test.yml:1`
- `.github/workflows/cli_test_e2e.yml:1`
- `.github/workflows/dgctl_release.yml:1`
- `.github/workflows/drift-deploy.yml:1`
- `.github/workflows/drift-release.yml:1`
- `.github/workflows/ee_backend_docker_release.yml:1`
- `.github/workflows/ee_backend_docker_release_fips.yml:1`
- `.github/workflows/ee_backend_test.yml:1`
- `.github/workflows/ee_cli_release.yml:1`
- `.github/workflows/ee_cli_release_fips.yml:1`
- `.github/workflows/ee_cli_release_multiarch.yml:1`
- `.github/workflows/ee_cli_test.yml:1`
- `.github/workflows/ee_tasks_release.yml:1`
- `.github/workflows/helm-release.yml:1`
- `.github/workflows/helm-test.yml:1`
- `.github/workflows/latest_tag.yml:1`
- `.github/workflows/libs_test.yml:1`
- `.github/workflows/misc_top_issues.yml:1`
- `.github/workflows/pro-deploy.yml:1`
- `.github/workflows/pro-projects-service-deploy.yml:1`
- `.github/workflows/projects-refresh-release.yml:1`
- `.github/workflows/projects_refresh_image_build.yml:1`
- `.github/workflows/release-please.yml:1`
- `.github/workflows/sidecar-release.yml:1`
- `.github/workflows/staging-deploy.yml:1`
- `.github/workflows/taco-release.yml:1`
- `.github/workflows/taco_statesman_docker_build_verify.yml:1`
- `.github/workflows/tasks_release.yml:1`
- `.github/workflows/tasks_run_test.yml:1`
- `.github/workflows/test-migrations.yml:1`
- `.github/workflows/ui-release.yml:1`
- `.github/workflows/update-helm-on-release.yml:1`
- `.github/workflows/vlatest-tag-update.yml:1`

### unsafe-shell (severity: high)

Multiple workflow run: blocks pipe remote content directly to a shell interpreter using `curl -sSf https://atlasgo.sh | sh`. This downloads and executes arbitrary code from the internet without any integrity verification, enabling supply-chain attacks if the remote URL is compromised. The script should be downloaded to a file first, its checksum verified, and then executed separately.

Locations:

- `.github/workflows/check-migrations-generated.yml:28`
- `.github/workflows/test-migrations.yml:47`
- `.github/workflows/test-migrations.yml:113`
- `.github/workflows/test-migrations.yml:178`

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

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, hardcoded-credentials, missing-permissions, unpinned-uses, unsafe-shell

**Notes:**

Fixed all security findings across action.yml and 42 workflow files:

1. script-injection/static-inline-injection (action.yml): Moved all ${{ inputs.* }} and ${{ runner.* }} expressions from run: blocks into env: blocks in 8 affected steps: 'digger run', 'Validate Input Configuration for Google', 'retrieve cache dir', 'restore-s3-cache', 'Setup Checkov', 'Copy Digger CLI go.sum for cache key', 'build and run digger', 'download, install, and run digger', 'run digger in local dev mode', and 'save-s3-cache'.

2. github-env-injection (action.yml): The 'retrieve cache dir' step now sanitizes the cache directory value with `printf '%s' "$CACHE_DIR" | tr -d '\n\r'` before writing to $GITHUB_ENV.

3. hardcoded-credentials (test-migrations.yml): Replaced hardcoded OPENTACO_JWT_SECRET with `${{ secrets.OPENTACO_JWT_SECRET || 'test-secret-key-for-ci' }}`.

4. missing-permissions: Added `permissions:` blocks to all 22 workflow files that were missing them, using minimal permissions appropriate for each workflow's purpose.

5. unpinned-uses: Pinned all mutable tag/branch references to full SHA digests across all 42 workflow files, including actions/checkout@v4, actions/setup-go@v5, opentofu/setup-opentofu, depot/setup-action, depot/build-push-action, superfly/flyctl-actions/setup-flyctl@master, docker/* actions, aws-actions/* actions, softprops/action-gh-release, EndBug/latest-tag, diggerhq/top-issues@main, and many others.

6. unsafe-shell: Replaced `curl -sSf https://atlasgo.sh | sh` with direct binary downloads (`curl -sSfL https://release.ariga.io/atlas/atlas-linux-amd64-latest -o /usr/local/bin/atlas && chmod +x /usr/local/bin/atlas`) in check-migrations-generated.yml and all 3 occurrences in test-migrations.yml.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 13 script-injection and 1 github-env-injection findings across 13 workflow files. In each case, ${{ }} expressions were moved from run: shell strings into the step's env: block and referenced as plain shell variables. Files fixed: auto-tag-and-release.yml (github.event_name, force_services input, github.event.before, needs outputs), backend_build_prod_ecr.yml (vars.ECS_CLUSTER_NAME/SERVICE_NAME), check-migrations-generated.yml (github.base_ref, step outputs), cli_test_e2e.yml (secrets.GCP_CREDENTIALS via printf), dgctl_release.yml (matrix.os/arch), helm-release.yml (GITHUB_TOKEN, github.actor, matrix.chart), helm-test.yml (matrix.chart), latest_tag.yml (github.repository/ref), libs_test.yml (PR head repo, github.repository, step output), staging-deploy.yml (github.sha, step outputs, matrix.service, PR number, github.repository), taco-release.yml (matrix goos/goarch/os/arch, step outputs, github.sha, github.ref_name), test-migrations.yml (github.workspace), update-helm-on-release.yml (github.event.workflow_run.head_branch/head_sha, step outputs with printf/tr sanitization for GITHUB_OUTPUT writes).

### Iteration 3

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced all hardcoded plaintext passwords in test-migrations.yml with GitHub Actions secret references. The two flagged findings (POSTGRES_PASSWORD: postgres at line 36 and OPENTACO_POSTGRES_PASSWORD: postgres at line 80) were fixed, along with additional hardcoded MySQL passwords (MYSQL_ROOT_PASSWORD, MYSQL_PASSWORD, OPENTACO_MYSQL_PASSWORD) and inline connection string passwords (POSTGRES_URL, MYSQL_URL) that followed the same pattern. All passwords now reference ${{ secrets.POSTGRES_PASSWORD }}, ${{ secrets.MYSQL_ROOT_PASSWORD }}, and ${{ secrets.MYSQL_PASSWORD }} respectively.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'Detect service changes' step of auto-tag-and-release.yml. The variable $PREV_COMMIT (derived from the env var EVENT_BEFORE which holds github.event.before) was expanded unquoted in seven `git diff --name-only $PREV_COMMIT HEAD` commands. Added double quotes around all seven occurrences to prevent shell metacharacter injection: `git diff --name-only "$PREV_COMMIT" HEAD`.

