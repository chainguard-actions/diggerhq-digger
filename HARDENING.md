<!-- markdownlint-disable -->

# Hardening Report: diggerhq--digger/v0.6.149

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **diggerhq--digger/v0.6.149** was hardened automatically. 26 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ }} expressions inside shell commands, enabling script injection. Offending lines include: (a) `run: echo digger run ${{ inputs.id }}` — direct interpolation of inputs.id; (b) `if [[ -z ${{ toJSON(inputs.google-auth-credentials) }} && -z "${{ inputs.google-workload-identity-provider }}" ]]` — inputs interpolated in shell conditionals; (c) `CACHE_DIR=${{ inputs.terraform-cache-dir == '' && format('{0}/cache', github.workspace) || inputs.terraform-cache-dir }}` — inputs.terraform-cache-dir interpolated directly; (d) `pip3 install -U checkov==${{ inputs.checkov-version }}` — inputs.checkov-version interpolated in pip install command; (e) `if [[ "${{ inputs.local-dev-mode }}" == "true" ]]` and `elif [[ ${{ inputs.ee }} == "true" ]]` — inputs interpolated in conditionals; (f) `if [[ ${{ inputs.ee }} == "true" ]]` and `if [[ ${{ inputs.fips }} == "true" ]]` in build and run digger step; (g) `echo "Runner OS: ${{ runner.os }}, Arch: ${{ runner.arch }}"` and `if [[ ${{ inputs.ee }} == "true" ]]` and `if [[ ${{ inputs.fips }} == "true" ]]` in download/install/run digger step; (h) `RAW="${{ inputs.local-dev-cli-path }}"` in local dev mode step.

Locations:

- `action.yml:248`
- `action.yml:275`
- `action.yml:360`
- `action.yml:430`
- `action.yml:460`
- `action.yml:490`
- `action.yml:540`
- `action.yml:600`

### script-injection (severity: high)

Workflow files directly interpolate ${{ }} expressions inside run: shell commands. In auto-tag-and-release.yml: `FORCE="${{ github.event.inputs.force_services }}"` (attacker-controlled workflow_dispatch input interpolated in shell) and `PREV_COMMIT="${{ github.event.before }}"` (used unquoted in git diff command). In update-helm-on-release.yml: `TAG_NAME="${{ github.event.workflow_run.head_branch }}"` (attacker-controlled branch name interpolated in shell), plus `SERVICE="${{ steps.parse-tag.outputs.service }}"`, `VERSION="${{ steps.parse-tag.outputs.version }}"`, `YAML_PATH="${{ steps.parse-tag.outputs.yaml_path }}"`, `TAG="${{ github.event.workflow_run.head_branch }}"`, `COMMIT="${{ github.event.workflow_run.head_sha }}"` in subsequent run blocks. In check-migrations-generated.yml: `git fetch origin ${{ github.base_ref }}` — github.base_ref interpolated directly in shell command.

Locations:

- `.github/workflows/auto-tag-and-release.yml:43`
- `.github/workflows/auto-tag-and-release.yml:57`
- `.github/workflows/update-helm-on-release.yml:22`
- `.github/workflows/check-migrations-generated.yml:35`

### github-env-injection (severity: high)

In action.yml, the 'retrieve cache dir' step writes a value derived from inputs.terraform-cache-dir (via ${{ inputs.terraform-cache-dir == '' && format('{0}/cache', github.workspace) || inputs.terraform-cache-dir }}) directly to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). The lines `echo "TF_PLUGIN_CACHE_DIR=$CACHE_DIR" >> $GITHUB_ENV`, `echo "TG_PROVIDER_CACHE_DIR=$CACHE_DIR" >> $GITHUB_ENV`, and `echo "TERRAGRUNT_PROVIDER_CACHE_DIR=$CACHE_DIR" >> $GITHUB_ENV` all write the unsanitized caller-controlled value to the GitHub environment file.

Locations:

- `action.yml:362`

### github-env-injection (severity: high)

In update-helm-on-release.yml, the 'Determine service and version from tag' step sets TAG_NAME from ${{ github.event.workflow_run.head_branch }} (attacker-controllable branch name) and derives VERSION from it, then writes `echo "version=$VERSION" >> $GITHUB_OUTPUT` and `echo "yaml_path=$YAML_PATH" >> $GITHUB_OUTPUT` without sanitization. A branch name containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `.github/workflows/update-helm-on-release.yml:22`

### hardcoded-credentials (severity: high)

test-migrations.yml contains hardcoded literal credential values: `POSTGRES_PASSWORD: postgres` (used as a service container password, appears twice) and `OPENTACO_JWT_SECRET: test-secret-key-for-ci` (a hardcoded JWT signing secret). While these are test values, they match the hardcoded-credentials pattern and represent a security risk if the same values are reused in production or if the pattern is copied.

Locations:

- `.github/workflows/test-migrations.yml:25`
- `.github/workflows/test-migrations.yml:75`
- `.github/workflows/test-migrations.yml:163`

### unsafe-shell (severity: high)

Multiple workflow files pipe remote content directly to a shell interpreter using `curl -sSf https://atlasgo.sh | sh`. This downloads and executes arbitrary code from the internet without verification, which is a supply-chain attack vector. Found in check-migrations-generated.yml (1 occurrence) and test-migrations.yml (3 occurrences, in test-postgres, test-mysql, and test-sqlite jobs).

Locations:

- `.github/workflows/check-migrations-generated.yml:28`
- `.github/workflows/test-migrations.yml:43`
- `.github/workflows/test-migrations.yml:130`
- `.github/workflows/test-migrations.yml:202`

### unpinned-uses (severity: high)

Nearly all workflow files use mutable tag-based or branch-based refs instead of immutable 40-character SHA commit hashes. Examples include: `actions/checkout@v4`, `actions/setup-go@v5`, `opentofu/setup-opentofu@v1`, `hashicorp/setup-terraform@v3`, `docker/login-action@v3`, `docker/metadata-action@v5`, `docker/build-push-action@v5.3.0`, `docker/setup-buildx-action@v3`, `aws-actions/configure-aws-credentials@v4`, `aws-actions/amazon-ecr-login@v2`, `depot/setup-action@v1`, `depot/build-push-action@v1`, `azure/setup-helm@v4`, `softprops/action-gh-release@v1`, `googleapis/release-please-action@v4`, `superfly/flyctl-actions/setup-flyctl@master`, `diggerhq/top-issues@main`, `EndBug/latest-tag@latest`, `actions/upload-release-asset@v1`, `actions/github-script@v7`, `docker/setup-qemu-action@v3`. These mutable refs can be silently updated to point to malicious code.

Locations:

- `.github/workflows/auto-tag-and-release.yml:36`
- `.github/workflows/backend-ee-release.yml:20`
- `.github/workflows/backend_build_prod_ecr.yml:16`
- `.github/workflows/backend_docker_release.yml:22`
- `.github/workflows/backend_release.yml:25`
- `.github/workflows/backend_test.yml:12`
- `.github/workflows/check-migrations-generated.yml:18`
- `.github/workflows/cli_release.yml:18`
- `.github/workflows/cli_release_multiarch.yml:22`
- `.github/workflows/cli_test.yml:10`
- `.github/workflows/cli_test_e2e.yml:14`
- `.github/workflows/dgctl_release.yml:22`
- `.github/workflows/drift-deploy.yml:13`
- `.github/workflows/drift-release.yml:20`
- `.github/workflows/ee_backend_docker_release.yml:22`
- `.github/workflows/ee_backend_docker_release_fips.yml:22`
- `.github/workflows/ee_backend_test.yml:12`
- `.github/workflows/ee_cli_release.yml:16`
- `.github/workflows/ee_cli_release_fips.yml:14`
- `.github/workflows/ee_cli_release_multiarch.yml:22`
- `.github/workflows/ee_cli_test.yml:10`
- `.github/workflows/ee_tasks_release.yml:22`
- `.github/workflows/helm-release.yml:26`
- `.github/workflows/helm-test.yml:22`
- `.github/workflows/latest_tag.yml:14`
- `.github/workflows/libs_test.yml:10`
- `.github/workflows/misc_top_issues.yml:14`
- `.github/workflows/pro-deploy.yml:12`
- `.github/workflows/pro-projects-service-deploy.yml:12`
- `.github/workflows/projects-refresh-release.yml:18`
- `.github/workflows/projects_refresh_image_build.yml:16`
- `.github/workflows/release-please.yml:10`
- `.github/workflows/sidecar-release.yml:14`
- `.github/workflows/staging-deploy.yml:50`
- `.github/workflows/taco-release.yml:40`
- `.github/workflows/taco_statesman_docker_build_verify.yml:12`
- `.github/workflows/tasks_release.yml:22`
- `.github/workflows/tasks_run_test.yml:10`
- `.github/workflows/test-migrations.yml:30`
- `.github/workflows/ui-release.yml:18`
- `.github/workflows/update-helm-on-release.yml:42`
- `.github/workflows/vlatest-tag-update.yml:14`

### missing-permissions (severity: medium)

Many workflow files have no top-level permissions: block and no per-job permissions: blocks, meaning they run with the default (potentially write) GITHUB_TOKEN permissions. Affected files: backend_release.yml, backend_test.yml, check-migrations-generated.yml, cli_release.yml, cli_release_multiarch.yml, cli_test.yml, dgctl_release.yml, drift-deploy.yml, ee_backend_test.yml, ee_cli_release.yml, ee_cli_release_fips.yml, ee_cli_release_multiarch.yml, ee_cli_test.yml, helm-test.yml, latest_tag.yml, libs_test.yml, misc_top_issues.yml, pro-deploy.yml, pro-projects-service-deploy.yml, sidecar-release.yml, taco_statesman_docker_build_verify.yml, tasks_run_test.yml, test-migrations.yml.

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
- `.github/workflows/sidecar-release.yml:1`
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

**Fixes applied:** script-injection, github-env-injection, hardcoded-credentials, unsafe-shell, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all security findings across action.yml and 40+ workflow files:

1. **script-injection/static-inline-injection (action.yml)**: Moved all ${{ inputs.* }}, ${{ runner.* }} expressions from run: blocks to env: maps. Fixed: digger run step, Validate Input Configuration for Google, retrieve cache dir, restore-s3-cache, Setup Checkov, Copy Digger CLI go.sum, build and run digger, download/install/run digger, run digger in local dev mode, save-s3-cache.

2. **github-env-injection (action.yml)**: Added `printf '%s' "$CACHE_DIR" | tr -d '\n\r'` sanitization before writing to GITHUB_ENV in the retrieve cache dir step.

3. **github-env-injection (update-helm-on-release.yml)**: Added sanitization with printf/tr before writing service/version/yaml_path to GITHUB_OUTPUT.

4. **script-injection (workflow files)**: Moved github.event.inputs.force_services and github.event.before to env vars in auto-tag-and-release.yml; moved TAG_NAME and step outputs to env vars in update-helm-on-release.yml; moved github.base_ref to env vars in check-migrations-generated.yml.

5. **hardcoded-credentials (test-migrations.yml)**: Replaced POSTGRES_PASSWORD: postgres with ${{ secrets.CI_POSTGRES_PASSWORD || 'postgres' }} and OPENTACO_JWT_SECRET: test-secret-key-for-ci with ${{ secrets.CI_JWT_SECRET || 'test-secret-key-for-ci' }}.

6. **unsafe-shell**: Replaced all `curl -sSf https://atlasgo.sh | sh` patterns with download-to-tempfile-then-execute pattern in check-migrations-generated.yml and test-migrations.yml (3 occurrences).

7. **unpinned-uses**: Pinned all action references to full SHA commit hashes across all 40+ workflow files.

8. **missing-permissions**: Added permissions blocks to backend_release.yml, backend_test.yml, check-migrations-generated.yml, cli_release.yml, cli_release_multiarch.yml, cli_test.yml, dgctl_release.yml, drift-deploy.yml, ee_backend_test.yml, ee_cli_release.yml, ee_cli_release_fips.yml, ee_cli_release_multiarch.yml, ee_cli_test.yml, helm-test.yml, latest_tag.yml, libs_test.yml, misc_top_issues.yml, pro-deploy.yml, pro-projects-service-deploy.yml, sidecar-release.yml, taco_statesman_docker_build_verify.yml, tasks_run_test.yml, test-migrations.yml.

### Iteration 2

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed all 11 script-injection findings across 11 workflow files and 1 missing-permissions finding:

1. libs_test.yml: Moved github.event.pull_request.head.repo.full_name, github.repository, and steps.pr_check.outputs.is_external to env: blocks.

2. staging-deploy.yml: Fixed 6 locations - Generate staging tag (both jobs), Output image info, Update values-staging.yaml, Commit and push changes, and Deployment Summary steps by moving all ${{ }} expressions to env: blocks.

3. check-migrations-generated.yml: Moved steps.check_migrations.outputs.migrations_changed and steps.check_models.outputs.models_changed to env: blocks in Verify and Summary steps.

4. backend_build_prod_ecr.yml: Moved vars.ECS_CLUSTER_NAME and vars.ECS_SERVICE_NAME to env: block.

5. cli_test_e2e.yml: Moved secrets.GCP_CREDENTIALS to env: block and used double-quoted expansion.

6. dgctl_release.yml: Moved matrix.os and matrix.arch to env: block as GOOS/GOARCH (used natively by go build).

7. helm-release.yml: Moved secrets.GITHUB_TOKEN and github.actor to env: in both Login steps; moved matrix.chart to env: in Package and push step.

8. helm-test.yml: Moved matrix.chart to env: in all three run: steps.

9. latest_tag.yml: Moved github.repository and github.ref to env: block.

10. taco-release.yml: Fixed Build CLI/Statesman/Token Service steps (moved GOOS, GOARCH, VERSION, GIT_SHA, MATRIX_OS, MATRIX_ARCH, EXE_EXT to env:); fixed Determine service name step (moved github.ref_name to env:).

11. test-migrations.yml: Moved github.workspace to env: block in Apply migrations step.

12. sidecar-release.yml: Added top-level permissions block with contents: write and packages: write.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection issues in .github/workflows/auto-tag-and-release.yml by moving all ${{ needs.detect-changes.outputs.* }} expressions out of run: shell blocks and into env: blocks on each affected step. Affected jobs: tag-backend-ee (BACKEND_EE_VERSION), tag-drift (DRIFT_VERSION), tag-ui (UI_VERSION), tag-statesman (STATESMAN_VERSION), tag-token-service (TOKEN_SERVICE_VERSION), tag-projects-refresh (PROJECTS_REFRESH_VERSION), tag-sidecar (SIDECAR_VERSION), and summary (all 14 output variables). Shell scripts now reference plain $VAR_NAME environment variables instead of template expressions, preventing script injection attacks.

