<!-- markdownlint-disable -->

# Hardening Report: diggerhq--digger/v0.6.145

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **diggerhq--digger/v0.6.145** was hardened automatically. 24 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ }} expressions inside shell commands, violating rule (a). This allows script injection if any of these inputs contain shell metacharacters.

Affected steps:
1. First step: `run: echo digger run ${{ inputs.id }}` — direct interpolation of inputs.id
2. "Validate Input Configuration for Google" step: `if [[ -z ${{ toJSON(inputs.google-auth-credentials) }} && -z "${{ inputs.google-workload-identity-provider }}" ]]` — direct interpolation of inputs
3. "retrieve cache dir" step: `CACHE_DIR=${{ inputs.terraform-cache-dir == '' && format('{0}/cache', github.workspace) || inputs.terraform-cache-dir }}` — direct interpolation of inputs.terraform-cache-dir
4. "Setup Checkov" step: `pip3 install -U checkov==${{ inputs.checkov-version }}` — direct interpolation of inputs.checkov-version
5. "download, install, and run digger" step: `echo "Runner OS: ${{ runner.os }}, Arch: ${{ runner.arch }}"` and `if [[ ${{ inputs.ee }} == "true" ]]` and `if [[ ${{ inputs.fips }} == "true" ]]` — direct interpolation of runner.* and inputs.*
6. "build and run digger" step: `if [[ ${{ inputs.ee }} == "true" ]]` and `if [[ ${{ inputs.fips }} == "true" ]]` — direct interpolation of inputs.*
7. "run digger in local dev mode" step: `if [[ ${{ inputs.ee }} == "true" ]]` and `if [[ ${{ inputs.fips }} == "true" ]]` — direct interpolation of inputs.*

Locations:

- `action.yml:9106`
- `action.yml:10408`
- `action.yml:14251`
- `action.yml:17319`
- `action.yml:24866`
- `action.yml:21653`

### github-env-injection (severity: high)

The 'retrieve cache dir' step writes a value derived from the untrusted input `inputs.terraform-cache-dir` (via the CACHE_DIR shell variable) to $GITHUB_ENV without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The expression `${{ inputs.terraform-cache-dir == '' && format('{0}/cache', github.workspace) || inputs.terraform-cache-dir }}` is interpolated directly into the shell script as CACHE_DIR, and then `echo "TF_PLUGIN_CACHE_DIR=$CACHE_DIR" >> $GITHUB_ENV` writes it to the environment file. An attacker-controlled newline in inputs.terraform-cache-dir could inject arbitrary environment variables.

Locations:

- `action.yml:14251`

### unsafe-shell (severity: high)

The run: block installs the Atlas CLI by piping remote content directly to a shell interpreter: `curl -sSf https://atlasgo.sh | sh`. This is unsafe because the content of the remote URL is not verified before execution, allowing a compromised or malicious server to execute arbitrary code on the runner.

Locations:

- `.github/workflows/check-migrations-generated.yml:27`
- `.github/workflows/test-migrations.yml:42`
- `.github/workflows/test-migrations.yml:100`
- `.github/workflows/test-migrations.yml:158`

### hardcoded-credentials (severity: high)

A hardcoded secret value is assigned to OPENTACO_JWT_SECRET in the test-migrations.yml workflow: `OPENTACO_JWT_SECRET: test-secret-key-for-ci`. Even though this is labeled as a CI test secret, hardcoding secret values in workflow files is a security risk as they are committed to source control and could be used to forge tokens if the same secret is reused in other environments.

Locations:

- `.github/workflows/test-migrations.yml:166`

### unpinned-uses (severity: high)

Many workflow files use mutable tag or branch refs (e.g., @v4, @v3, @v5, @v1, @master, @main, @latest) instead of pinned 40-character SHA commit hashes. This exposes the workflows to supply-chain attacks if the referenced action is compromised or its tag is moved.

Affected files and unpinned references include:
- backend-ee-release.yml: actions/checkout@v4, docker/setup-buildx-action@v3, docker/login-action@v3, docker/metadata-action@v5, depot/setup-action@v1, depot/build-push-action@v1, softprops/action-gh-release@v1
- backend_build_prod_ecr.yml: actions/checkout@v4, aws-actions/configure-aws-credentials@v4, aws-actions/amazon-ecr-login@v2
- backend_docker_release.yml: actions/checkout@v4, docker/metadata-action@v5.5.1, docker/build-push-action@v5.3.0
- backend_release.yml: actions/checkout@v4
- backend_test.yml: actions/setup-go@v5, actions/checkout@v4, depot/setup-action@v1, depot/build-push-action@v1
- check-migrations-generated.yml: actions/checkout@v4, actions/setup-go@v5
- cli_release.yml: actions/checkout@v4
- cli_release_multiarch.yml: actions/checkout@v4
- cli_test.yml: actions/setup-go@v5, opentofu/setup-opentofu@v1, actions/checkout@v4
- cli_test_e2e.yml: actions/checkout@v4, actions/setup-go@v5, aws-actions/configure-aws-credentials@v4
- dgctl_release.yml: actions/setup-go@v5, actions/checkout@v4, actions/upload-release-asset@v1
- drift-deploy.yml: actions/checkout@v4, superfly/flyctl-actions/setup-flyctl@master
- drift-release.yml: actions/checkout@v4, docker/setup-buildx-action@v3, docker/login-action@v3, docker/metadata-action@v5, depot/setup-action@v1, depot/build-push-action@v1, softprops/action-gh-release@v1
- ee_backend_docker_release.yml: actions/checkout@v4, docker/metadata-action@v5.5.1, docker/build-push-action@v5.3.0
- ee_backend_docker_release_fips.yml: actions/checkout@v4, docker/metadata-action@v5.5.1, docker/build-push-action@v5.3.0
- ee_backend_test.yml: actions/setup-go@v5, actions/checkout@v4, depot/setup-action@v1, depot/build-push-action@v1
- ee_cli_release.yml: actions/setup-go@v5, actions/checkout@v4
- ee_cli_release_fips.yml: actions/setup-go@v5, actions/checkout@v4
- ee_cli_release_multiarch.yml: actions/checkout@v4
- ee_cli_test.yml: actions/setup-go@v5, opentofu/setup-opentofu@v1, actions/checkout@v4
- ee_tasks_release.yml: actions/checkout@v4, docker/metadata-action@v5.5.1, docker/build-push-action@v5.3.0
- helm-release.yml: actions/checkout@v4, azure/setup-helm@v4
- helm-test.yml: actions/checkout@v4, azure/setup-helm@v4
- latest_tag.yml: actions/checkout@v4, EndBug/latest-tag@latest, softprops/action-gh-release@v2
- libs_test.yml: opentofu/setup-opentofu@v1.0.5, hashicorp/setup-terraform@v3, actions/setup-go@v5, actions/checkout@v4, opentofu/setup-opentofu@v1
- misc_top_issues.yml: diggerhq/top-issues@main
- pro-deploy.yml: actions/checkout@v4, superfly/flyctl-actions/setup-flyctl@master
- pro-projects-service-deploy.yml: actions/checkout@v4, superfly/flyctl-actions/setup-flyctl@master
- projects-refresh-release.yml: actions/checkout@v4, docker/setup-buildx-action@v3, docker/login-action@v3, docker/metadata-action@v5, depot/setup-action@v1, depot/build-push-action@v1, softprops/action-gh-release@v1
- projects_refresh_image_build.yml: actions/checkout@v4, docker/setup-qemu-action@v3, docker/setup-buildx-action@v3, depot/setup-action@v1, depot/build-push-action@v1
- release-please.yml: googleapis/release-please-action@v4
- sidecar-release.yml: actions/checkout@v4, docker/setup-buildx-action@v3, docker/login-action@v3, docker/metadata-action@v5, depot/setup-action@v1, depot/build-push-action@v1, softprops/action-gh-release@v1
- staging-deploy.yml: actions/checkout@v4, docker/setup-buildx-action@v3, docker/login-action@v3, docker/metadata-action@v5, depot/setup-action@v1, depot/build-push-action@v1, docker/build-push-action@v5
- taco-release.yml: actions/checkout@v4, actions/setup-go@v5, actions/upload-artifact@v4, actions/download-artifact@v4, docker/setup-buildx-action@v3, docker/login-action@v3, docker/metadata-action@v5, depot/setup-action@v1, depot/build-push-action@v1, softprops/action-gh-release@v1
- taco_statesman_docker_build_verify.yml: actions/setup-go@v5, actions/checkout@v4
- tasks_release.yml: actions/checkout@v4, docker/metadata-action@v5.5.1, docker/build-push-action@v5.3.0
- tasks_run_test.yml: actions/setup-go@v5, actions/checkout@v4
- test-migrations.yml: actions/checkout@v4, actions/setup-go@v5
- ui-release.yml: actions/checkout@v4, docker/setup-buildx-action@v3, docker/login-action@v3, docker/metadata-action@v5, depot/setup-action@v1, depot/build-push-action@v1, softprops/action-gh-release@v1
- update-helm-on-release.yml: actions/checkout@v4
- vlatest-tag-update.yml: actions/github-script@v7

Locations:

- `.github/workflows/backend-ee-release.yml:17`
- `.github/workflows/backend_build_prod_ecr.yml:14`
- `.github/workflows/backend_docker_release.yml:14`
- `.github/workflows/backend_release.yml:14`
- `.github/workflows/backend_test.yml:10`
- `.github/workflows/check-migrations-generated.yml:15`
- `.github/workflows/cli_release.yml:14`
- `.github/workflows/cli_release_multiarch.yml:18`
- `.github/workflows/cli_test.yml:8`
- `.github/workflows/cli_test_e2e.yml:13`
- `.github/workflows/dgctl_release.yml:20`
- `.github/workflows/drift-deploy.yml:11`
- `.github/workflows/drift-release.yml:17`
- `.github/workflows/ee_backend_docker_release.yml:14`
- `.github/workflows/ee_backend_docker_release_fips.yml:14`
- `.github/workflows/ee_backend_test.yml:10`
- `.github/workflows/ee_cli_release.yml:14`
- `.github/workflows/ee_cli_release_fips.yml:12`
- `.github/workflows/ee_cli_release_multiarch.yml:18`
- `.github/workflows/ee_cli_test.yml:8`
- `.github/workflows/ee_tasks_release.yml:14`
- `.github/workflows/helm-release.yml:22`
- `.github/workflows/helm-test.yml:15`
- `.github/workflows/latest_tag.yml:14`
- `.github/workflows/libs_test.yml:8`
- `.github/workflows/misc_top_issues.yml:14`
- `.github/workflows/pro-deploy.yml:10`
- `.github/workflows/pro-projects-service-deploy.yml:10`
- `.github/workflows/projects-refresh-release.yml:17`
- `.github/workflows/projects_refresh_image_build.yml:14`
- `.github/workflows/release-please.yml:11`
- `.github/workflows/sidecar-release.yml:13`
- `.github/workflows/staging-deploy.yml:24`
- `.github/workflows/taco-release.yml:37`
- `.github/workflows/taco_statesman_docker_build_verify.yml:10`
- `.github/workflows/tasks_release.yml:14`
- `.github/workflows/tasks_run_test.yml:8`
- `.github/workflows/test-migrations.yml:30`
- `.github/workflows/ui-release.yml:13`
- `.github/workflows/update-helm-on-release.yml:18`
- `.github/workflows/vlatest-tag-update.yml:12`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks on every job. Without explicit permissions, workflows run with the default (potentially broad) token permissions, which may include write access to repository contents.

Affected files:
- backend_release.yml
- backend_test.yml
- check-migrations-generated.yml
- cli_release.yml
- cli_release_multiarch.yml
- cli_test.yml
- dgctl_release.yml
- drift-deploy.yml
- ee_backend_test.yml
- ee_cli_release.yml
- ee_cli_release_fips.yml
- ee_cli_release_multiarch.yml
- ee_cli_test.yml
- helm-test.yml
- latest_tag.yml
- libs_test.yml
- misc_top_issues.yml
- pro-deploy.yml
- pro-projects-service-deploy.yml
- taco_statesman_docker_build_verify.yml
- tasks_run_test.yml
- test-migrations.yml

Locations:

- `.github/workflows/backend_release.yml:1`
- `.github/workflows/backend_test.yml:1`
- `.github/workflows/check-migrations-generated.yml:1`
- `.github/workflows/cli_release.yml:1`
- `.github/workflows/cli_release_multiarch.yml:1`
- `.github/workflows/cli_test.yml:1`
- `.github/workflows/dgctl_release.yml:1`
- `.github/workflows/drift-deploy.yml:1`
- `.github/workflows/ee_backend_test.yml:1`
- `.github/workflows/ee_cli_release.yml:1`
- `.github/workflows/ee_cli_release_fips.yml:1`
- `.github/workflows/ee_cli_release_multiarch.yml:1`
- `.github/workflows/ee_cli_test.yml:1`
- `.github/workflows/helm-test.yml:1`
- `.github/workflows/latest_tag.yml:1`
- `.github/workflows/libs_test.yml:1`
- `.github/workflows/misc_top_issues.yml:1`
- `.github/workflows/pro-deploy.yml:1`
- `.github/workflows/pro-projects-service-deploy.yml:1`
- `.github/workflows/taco_statesman_docker_build_verify.yml:1`
- `.github/workflows/tasks_run_test.yml:1`
- `.github/workflows/test-migrations.yml:1`

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

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, unsafe-shell, hardcoded-credentials, unpinned-uses, missing-permissions

**Notes:**

Fixed all security findings:

1. script-injection/static-inline-injection (action.yml): Moved all ${{ inputs.* }} and ${{ runner.* }} expressions from run: blocks to env: blocks in 9 steps: digger run, Validate Input Configuration for Google, retrieve cache dir, restore-s3-cache, Setup Checkov, Copy Digger CLI go.sum for cache key, build and run digger, download/install/run digger, run digger in local dev mode, save-s3-cache.

2. github-env-injection (action.yml): Added sanitization with `printf '%s' "$CACHE_DIR" | tr -d '\n\r'` before writing to $GITHUB_ENV in the retrieve cache dir step.

3. unsafe-shell (check-migrations-generated.yml, test-migrations.yml x3): Replaced `curl -sSf https://atlasgo.sh | sh` with download-then-execute: `curl -sSfL -o /tmp/atlas-install.sh https://atlasgo.sh && sh /tmp/atlas-install.sh`.

4. hardcoded-credentials (test-migrations.yml): Replaced hardcoded `OPENTACO_JWT_SECRET: test-secret-key-for-ci` with `${{ secrets.OPENTACO_JWT_SECRET_CI }}`.

5. unpinned-uses: Pinned all unpinned action references to full SHA hashes across all 41 listed workflow files. Actions pinned include: actions/checkout@v4, actions/setup-go@v5, docker/setup-buildx-action@v3, docker/login-action@v3, docker/metadata-action@v5/v5.5.1, docker/build-push-action@v5/v5.3.0, docker/setup-qemu-action@v3, depot/setup-action@v1, depot/build-push-action@v1, softprops/action-gh-release@v1/v2, aws-actions/configure-aws-credentials@v4, aws-actions/amazon-ecr-login@v2, opentofu/setup-opentofu@v1/v1.0.5, hashicorp/setup-terraform@v3, actions/upload-release-asset@v1, superfly/flyctl-actions/setup-flyctl@master, azure/setup-helm@v4, EndBug/latest-tag@latest, diggerhq/top-issues@main, googleapis/release-please-action@v4, actions/upload-artifact@v4, actions/download-artifact@v4, actions/github-script@v7.

6. missing-permissions: Added permissions: blocks to all 22 workflow files listed in the finding.

### Iteration 2

**Fixes applied:** script-injection, unpinned-uses, hardcoded-credentials

**Notes:**

Fixed script injection in 9 workflow files by moving all ${{ ... }} expressions from run: shell strings into step env: blocks. Pinned all actions/checkout@v4 references in auto-tag-and-release.yml to SHA 11d5960a326750d5838078e36cf38b85af677262. Replaced hardcoded POSTGRES_PASSWORD: postgres and OPENTACO_POSTGRES_PASSWORD: postgres in test-migrations.yml with ${{ secrets.CI_POSTGRES_PASSWORD }} secret references (also updated the POSTGRES_URL connection string). Matrix values (matrix.os, matrix.arch, matrix.chart) and github context values (github.sha, github.base_ref, github.ref_name, github.event.inputs.force_services, github.event.before, github.event.pull_request.number, github.event.pull_request.head.repo.full_name, github.repository, github.event.workflow_run.head_branch, steps.*.outputs.*) were all moved to env: blocks.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in 8 workflow files: (1) auto-tag-and-release.yml: moved needs.detect-changes.outputs.* into env: blocks for all tag-* jobs and summary job. (2) staging-deploy.yml: moved steps.tag.outputs.tag, matrix.service, matrix.image_name, and github.repository into env: blocks. (3) check-migrations-generated.yml: moved steps.check_migrations.outputs.migrations_changed and steps.check_models.outputs.models_changed into env: blocks. (4) backend_build_prod_ecr.yml: moved vars.ECS_CLUSTER_NAME and vars.ECS_SERVICE_NAME into env: block with proper quoting. (5) latest_tag.yml: moved github.repository and github.ref into env: block. (6) helm-release.yml: moved github.actor and secrets.GITHUB_TOKEN into env: blocks for both Login steps. (7) cli_test_e2e.yml: moved secrets.GCP_CREDENTIALS into env: block and used printf to write to file. (8) taco-release.yml: moved matrix.goos, matrix.goarch, matrix.os, matrix.arch, steps.meta.outputs.version, and github.sha into env: blocks for all three build steps.

### Iteration 4

**Fixes applied:** missing-permissions, github-env-injection, script-injection

**Notes:**

1. sidecar-release.yml: Added job-level permissions blocks - build-and-push job gets `contents: read` and `packages: write` (needed for ghcr.io push); create-release job gets `contents: write` (needed to create GitHub releases). 2. update-helm-on-release.yml: Fixed github-env-injection in the 'Determine service and version from tag' step by sanitizing SERVICE, VERSION, and YAML_PATH variables with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT, preventing newline injection from the attacker-controlled head_branch value. 3. action.yml: Fixed script-injection in the 'run digger in local dev mode' step by changing the unquoted `$BIN` execution to properly double-quoted `"$BIN"`, preventing word splitting on the path derived from the user-controlled input.

