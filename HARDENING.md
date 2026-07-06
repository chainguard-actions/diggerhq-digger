<!-- markdownlint-disable -->

# Hardening Report: diggerhq--digger--/v0.6.147

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **diggerhq--digger--/v0.6.147** was hardened automatically. 24 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ }}` expressions inside shell commands, violating rule (a). Affected steps:
1. 'digger run' step: `run: echo digger run ${{ inputs.id }}` — direct injection of `inputs.id`.
2. 'Validate Input Configuration for Google' step: `${{ inputs.google-workload-identity-provider }}` and `${{ inputs.google-service-account }}` interpolated directly in shell conditionals.
3. 'retrieve cache dir' step: `CACHE_DIR=${{ inputs.terraform-cache-dir == '' && ... }}` — expression interpolated directly into shell assignment.
4. 'restore-s3-cache' step: `BUCKET="${{ inputs.cache-dependencies-s3-bucket }}"`, `REGION="${{ inputs.cache-dependencies-s3-region }}"`, `PREFIX="${{ inputs.cache-dependencies-s3-bucket-prefix }}"`, `SCRIPT_PATH="${{ github.action_path }}/..."` — all interpolated directly in shell.
5. 'Setup Checkov' step: `pip3 install -U checkov==${{ inputs.checkov-version }}` — direct injection.
6. 'Copy Digger CLI go.sum' step: `${{ inputs.local-dev-mode }}` and `${{ inputs.ee }}` interpolated in shell conditionals.
7. 'build and run digger' step: `${{ inputs.ee }}` and `${{ inputs.fips }}` interpolated in shell conditionals.
8. 'download, install, and run digger' step: `${{ runner.os }}`, `${{ runner.arch }}`, `${{ inputs.ee }}`, `${{ inputs.fips }}` interpolated in shell.
9. 'run digger in local dev mode' step: `${{ inputs.local-dev-cli-path }}` interpolated in shell.
10. 'save-s3-cache' step: `${{ inputs.cache-dependencies-s3-bucket }}`, `${{ inputs.cache-dependencies-s3-region }}`, `${{ inputs.cache-dependencies-s3-bucket-prefix }}`, `${{ github.action_path }}` interpolated in shell.

Locations:

- `action.yml:237`
- `action.yml:263`
- `action.yml:290`
- `action.yml:310`
- `action.yml:360`
- `action.yml:380`
- `action.yml:400`
- `action.yml:430`
- `action.yml:460`

### github-env-injection (severity: high)

The 'retrieve cache dir' step in action.yml directly interpolates `${{ inputs.terraform-cache-dir }}` (via a conditional expression) into a shell variable `CACHE_DIR`, then writes it to `$GITHUB_ENV` without sanitization (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled `terraform-cache-dir` input containing newlines could inject arbitrary environment variables. The offending code: `CACHE_DIR=${{ inputs.terraform-cache-dir == '' && format('{0}/cache', github.workspace) || inputs.terraform-cache-dir }}` followed by `echo "TF_PLUGIN_CACHE_DIR=$CACHE_DIR" >> $GITHUB_ENV`.

Locations:

- `action.yml:290`

### unpinned-uses (severity: high)

Many workflow files use mutable tag or version refs instead of full 40-character SHA commit pins. Unpinned references found across multiple files include: `actions/checkout@v4`, `actions/setup-go@v5`, `docker/setup-buildx-action@v3`, `docker/login-action@v3`, `docker/metadata-action@v5`, `docker/metadata-action@v5.5.1`, `docker/build-push-action@v5.3.0`, `docker/build-push-action@v5`, `docker/setup-qemu-action@v3`, `depot/setup-action@v1`, `depot/build-push-action@v1`, `softprops/action-gh-release@v1`, `softprops/action-gh-release@v2`, `EndBug/latest-tag@latest`, `diggerhq/top-issues@main`, `superfly/flyctl-actions/setup-flyctl@master`, `aws-actions/configure-aws-credentials@v4`, `aws-actions/amazon-ecr-login@v2`, `opentofu/setup-opentofu@v1`, `opentofu/setup-opentofu@v1.0.5`, `hashicorp/setup-terraform@v3`, `azure/setup-helm@v4`, `googleapis/release-please-action@v4`, `actions/upload-artifact@v4`, `actions/download-artifact@v4`, `actions/github-script@v7`, `actions/upload-release-asset@v1`.

Locations:

- `.github/workflows/auto-tag-and-release.yml:35`
- `.github/workflows/backend-ee-release.yml:17`
- `.github/workflows/backend_build_prod_ecr.yml:14`
- `.github/workflows/backend_docker_release.yml:22`
- `.github/workflows/backend_release.yml:24`
- `.github/workflows/backend_test.yml:13`
- `.github/workflows/check-migrations-generated.yml:18`
- `.github/workflows/cli_release.yml:17`
- `.github/workflows/cli_release_multiarch.yml:24`
- `.github/workflows/cli_test.yml:10`
- `.github/workflows/cli_test_e2e.yml:14`
- `.github/workflows/dgctl_release.yml:22`
- `.github/workflows/drift-deploy.yml:11`
- `.github/workflows/drift-release.yml:18`
- `.github/workflows/ee_backend_docker_release.yml:22`
- `.github/workflows/ee_backend_docker_release_fips.yml:22`
- `.github/workflows/ee_backend_test.yml:13`
- `.github/workflows/ee_cli_release.yml:17`
- `.github/workflows/ee_cli_release_fips.yml:14`
- `.github/workflows/ee_cli_release_multiarch.yml:24`
- `.github/workflows/ee_cli_test.yml:10`
- `.github/workflows/ee_tasks_release.yml:22`
- `.github/workflows/helm-release.yml:27`
- `.github/workflows/helm-test.yml:19`
- `.github/workflows/latest_tag.yml:13`
- `.github/workflows/libs_test.yml:10`
- `.github/workflows/misc_top_issues.yml:13`
- `.github/workflows/pro-deploy.yml:10`
- `.github/workflows/pro-projects-service-deploy.yml:10`
- `.github/workflows/projects-refresh-release.yml:18`
- `.github/workflows/projects_refresh_image_build.yml:18`
- `.github/workflows/release-please.yml:12`
- `.github/workflows/sidecar-release.yml:13`
- `.github/workflows/staging-deploy.yml:55`
- `.github/workflows/taco-release.yml:40`
- `.github/workflows/taco_statesman_docker_build_verify.yml:13`
- `.github/workflows/tasks_release.yml:22`
- `.github/workflows/tasks_run_test.yml:10`
- `.github/workflows/test-migrations.yml:34`
- `.github/workflows/ui-release.yml:18`
- `.github/workflows/update-helm-on-release.yml:22`
- `.github/workflows/vlatest-tag-update.yml:13`

### missing-permissions (severity: medium)

Multiple workflow files have no top-level `permissions:` key and no job-level `permissions:` keys, meaning they run with the default (potentially broad) GITHUB_TOKEN permissions. Affected files: backend_release.yml, backend_test.yml, check-migrations-generated.yml, cli_release.yml, cli_release_multiarch.yml, cli_test.yml, dgctl_release.yml, drift-deploy.yml, ee_backend_test.yml, ee_cli_release.yml, ee_cli_release_fips.yml, ee_cli_release_multiarch.yml, ee_cli_test.yml, helm-test.yml, latest_tag.yml, libs_test.yml, misc_top_issues.yml, pro-deploy.yml, pro-projects-service-deploy.yml, sidecar-release.yml, taco_statesman_docker_build_verify.yml, tasks_run_test.yml, test-migrations.yml.

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

### unsafe-shell (severity: high)

Multiple workflow files pipe remote content directly to a shell interpreter using `curl -sSf https://atlasgo.sh | sh`. This downloads and executes arbitrary code from a remote server without verification, which is a supply-chain attack vector. The script should be downloaded to a file first, verified, and then executed separately.

Locations:

- `.github/workflows/check-migrations-generated.yml:28`
- `.github/workflows/test-migrations.yml:50`
- `.github/workflows/test-migrations.yml:100`
- `.github/workflows/test-migrations.yml:150`

### hardcoded-credentials (severity: high)

A hardcoded JWT secret value `test-secret-key-for-ci` is assigned to `OPENTACO_JWT_SECRET` in the test-migrations.yml workflow. Hardcoding secret values in workflow files is a security risk as the value is committed to source control and may be reused in other environments.

Locations:

- `.github/workflows/test-migrations.yml:175`

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

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, unsafe-shell, hardcoded-credentials, static-inline-injection

**Notes:**

Fixed all security findings: (1) script-injection/static-inline-injection in action.yml: moved all ${{ inputs.* }} and ${{ runner.* }} expressions from run: blocks into env: blocks for 10 affected steps (digger run, Validate Input Configuration for Google, retrieve cache dir, restore-s3-cache, Setup Checkov, Copy Digger CLI go.sum, build and run digger, download/install/run digger, run digger in local dev mode, save-s3-cache); (2) github-env-injection in action.yml: sanitized CACHE_DIR with 'printf | tr -d' before writing to GITHUB_ENV; (3) unsafe-shell in check-migrations-generated.yml and test-migrations.yml (3 occurrences): replaced 'curl -sSf https://atlasgo.sh | sh' with download-to-file-then-execute pattern; (4) hardcoded-credentials in test-migrations.yml: replaced hardcoded JWT secret 'test-secret-key-for-ci' with ${{ secrets.OPENTACO_JWT_SECRET_CI }}; (5) missing-permissions: added permissions blocks to 23 workflow files with appropriate minimal permissions (contents:read for test/build workflows, contents:write for release workflows, issues:write for misc_top_issues.yml, packages:write for container release workflows); (6) unpinned-uses: pinned all action references to full SHA commits across all workflow files including actions/checkout, actions/setup-go, docker/* actions, depot/* actions, softprops/action-gh-release, EndBug/latest-tag, diggerhq/top-issues, superfly/flyctl-actions, aws-actions/*, opentofu/setup-opentofu, hashicorp/setup-terraform, azure/setup-helm, googleapis/release-please-action, actions/upload-artifact, actions/download-artifact, actions/github-script, actions/upload-release-asset.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, hardcoded-credentials

**Notes:**

Fixed all three finding types across 8 workflow files:

1. script-injection: Moved all ${{ }} expressions from run: blocks to env: blocks in:
   - auto-tag-and-release.yml: github.event_name, github.event.inputs.force_services, github.event.before
   - check-migrations-generated.yml: github.base_ref, steps.check_migrations.outputs.migrations_changed, steps.check_models.outputs.models_changed
   - libs_test.yml: github.event.pull_request.head.repo.full_name, github.repository, steps.pr_check.outputs.is_external
   - latest_tag.yml: github.repository, github.ref
   - staging-deploy.yml: github.sha, steps.tag.outputs.tag, matrix.service, github.event.pull_request.number, github.repository
   - update-helm-on-release.yml: github.event.workflow_run.head_branch, steps.parse-tag.outputs.*, github.event.workflow_run.head_sha
   - taco-release.yml: matrix.goos, matrix.goarch, steps.meta.outputs.version, github.sha, matrix.os, matrix.arch, matrix.goos=='windows'&&'.exe'||'', github.ref_name
   - helm-release.yml: secrets.GITHUB_TOKEN, github.actor, matrix.chart

2. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before writing to GITHUB_OUTPUT in:
   - update-helm-on-release.yml: VERSION, SERVICE, YAML_PATH derived from github.event.workflow_run.head_branch
   - staging-deploy.yml: STAGING_TAG derived from github.sha

3. hardcoded-credentials: Replaced hardcoded 'postgres' password literals with ${{ secrets.TEST_POSTGRES_PASSWORD }} in test-migrations.yml (service container env, OPENTACO_POSTGRES_PASSWORD env, and POSTGRES_URL construction)

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/test-migrations.yml (test-sqlite job, 'Apply migrations manually' step). Replaced both occurrences of `${{ github.workspace }}` in the run: block with the standard `$GITHUB_WORKSPACE` environment variable, which holds the same value but avoids direct expression interpolation into shell commands.

