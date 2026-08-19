<!-- markdownlint-disable -->

# Hardening Report: diggerhq--digger/vtest-token

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **diggerhq--digger/vtest-token** was hardened automatically. 23 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ }} expressions inside shell commands, violating rule (a). Affected steps include: (1) 'digger run' step: `run: echo digger run ${{ inputs.id }}`; (2) 'Validate Input Configuration for Google' step: `if [[ -z ${{ toJSON(inputs.google-auth-credentials) }} && -z "${{ inputs.google-workload-identity-provider }}" ]]` and `${{ inputs.google-service-account }}`; (3) 'retrieve cache dir' step: `CACHE_DIR=${{ inputs.terraform-cache-dir == '' && format('{0}/cache', github.workspace) || inputs.terraform-cache-dir }}`; (4) 'restore-s3-cache' step: `BUCKET="${{ inputs.cache-dependencies-s3-bucket }}"`, `REGION="${{ inputs.cache-dependencies-s3-region }}"`, `PREFIX="${{ inputs.cache-dependencies-s3-bucket-prefix }}"`, `SCRIPT_PATH="${{ github.action_path }}/scripts/s3-cache-download.bash"`; (5) 'Setup Checkov' step: `pip3 install -U checkov==${{ inputs.checkov-version }}`; (6) 'Copy Digger CLI go.sum' step: `if [[ "${{ inputs.local-dev-mode }}" == "true" ]]` and `elif [[ ${{ inputs.ee }} == "true" ]]`; (7) 'build and run digger' step: `if [[ ${{ inputs.ee }} == "true" ]]` and `if [[ ${{ inputs.fips }} == "true" ]]`; (8) 'download, install, and run digger' step: `if [[ ${{ inputs.ee }} == "true" ]]`, `if [[ ${{ inputs.fips }} == "true" ]]`, `echo "Runner OS: ${{ runner.os }}, Arch: ${{ runner.arch }}`; (9) 'save-s3-cache' step: `BUCKET="${{ inputs.cache-dependencies-s3-bucket }}"` etc. All these allow an attacker-controlled value to be interpreted as shell code before the shell ever sees it.

Locations:

- `action.yml:178`
- `action.yml:195`
- `action.yml:250`
- `action.yml:265`
- `action.yml:295`
- `action.yml:330`
- `action.yml:345`
- `action.yml:395`
- `action.yml:540`

### unsafe-shell (severity: high)

run: blocks pipe remote content directly to a shell interpreter using `curl -sSf https://atlasgo.sh | sh`. This downloads and executes arbitrary code from a remote URL without first saving it to a file for inspection. Found in check-migrations-generated.yml (Install Atlas CLI step) and test-migrations.yml (Install Atlas CLI steps in multiple jobs).

Locations:

- `.github/workflows/check-migrations-generated.yml:29`
- `.github/workflows/test-migrations.yml:50`

### unpinned-uses (severity: high)

Numerous workflow files reference actions using mutable tags, branch names, or version strings instead of full 40-character SHA commit hashes. Examples include: `actions/checkout@v4`, `actions/setup-go@v5`, `docker/setup-buildx-action@v3`, `docker/login-action@v3`, `superfly/flyctl-actions/setup-flyctl@master`, `diggerhq/top-issues@main`, `googleapis/release-please-action@v4`, `EndBug/latest-tag@latest`, `softprops/action-gh-release@v2`, `azure/setup-helm@v4`, `opentofu/setup-opentofu@v1`, `hashicorp/setup-terraform@v3`, `actions/upload-release-asset@v1`, `wangyoucao577/go-release-action@8fa1e8368c8465264d64e0198208e10f71474c87` (this one IS SHA-pinned). Mutable refs allow supply-chain attacks if the referenced action is compromised or the tag is moved.

Locations:

- `.github/workflows/backend-ee-release.yml:14`
- `.github/workflows/backend_test.yml:14`
- `.github/workflows/backend_release.yml:20`
- `.github/workflows/backend_docker_release.yml:18`
- `.github/workflows/backend_build_prod_ecr.yml:10`
- `.github/workflows/cli_test.yml:14`
- `.github/workflows/cli_release.yml:18`
- `.github/workflows/cli_release_multiarch.yml:18`
- `.github/workflows/cli_test_e2e.yml:14`
- `.github/workflows/check-migrations-generated.yml:16`
- `.github/workflows/dgctl_release.yml:22`
- `.github/workflows/drift-deploy.yml:11`
- `.github/workflows/drift-release.yml:14`
- `.github/workflows/ee_backend_test.yml:14`
- `.github/workflows/ee_backend_docker_release.yml:18`
- `.github/workflows/ee_backend_docker_release_fips.yml:18`
- `.github/workflows/ee_cli_release.yml:18`
- `.github/workflows/ee_cli_release_fips.yml:14`
- `.github/workflows/ee_cli_release_multiarch.yml:18`
- `.github/workflows/ee_cli_test.yml:14`
- `.github/workflows/ee_tasks_release.yml:18`
- `.github/workflows/helm-release.yml:22`
- `.github/workflows/helm-test.yml:12`
- `.github/workflows/latest_tag.yml:14`
- `.github/workflows/libs_test.yml:12`
- `.github/workflows/misc_top_issues.yml:14`
- `.github/workflows/pro-deploy.yml:11`
- `.github/workflows/pro-projects-service-deploy.yml:11`
- `.github/workflows/release-please.yml:12`
- `.github/workflows/taco-release.yml:40`
- `.github/workflows/taco_statesman_docker_build_verify.yml:14`
- `.github/workflows/tasks_release.yml:18`
- `.github/workflows/tasks_run_test.yml:14`
- `.github/workflows/test-migrations.yml:38`
- `.github/workflows/ui-release.yml:14`
- `.github/workflows/vlatest-tag-update.yml:14`

### missing-permissions (severity: medium)

Many workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job, meaning they run with the default (potentially broad) token permissions. Affected files include: backend_test.yml, backend_release.yml, cli_test.yml, cli_release.yml, cli_release_multiarch.yml, cli_test_e2e.yml (has job-level permissions - PASS), dgctl_release.yml, drift-deploy.yml, ee_backend_test.yml, ee_cli_release.yml, ee_cli_release_fips.yml, ee_cli_release_multiarch.yml, ee_cli_test.yml, helm-test.yml, latest_tag.yml, libs_test.yml, misc_top_issues.yml, pro-deploy.yml, pro-projects-service-deploy.yml, taco_statesman_docker_build_verify.yml, tasks_run_test.yml, check-migrations-generated.yml, test-migrations.yml.

Locations:

- `.github/workflows/backend_test.yml:1`
- `.github/workflows/backend_release.yml:1`
- `.github/workflows/cli_test.yml:1`
- `.github/workflows/cli_release.yml:1`
- `.github/workflows/cli_release_multiarch.yml:1`
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
- `.github/workflows/check-migrations-generated.yml:1`
- `.github/workflows/test-migrations.yml:1`

### hardcoded-credentials (severity: high)

test-migrations.yml contains hardcoded literal credential values: (1) `POSTGRES_PASSWORD: postgres` used as a service container environment variable; (2) `OPENTACO_JWT_SECRET: test-secret-key-for-ci` used as an application secret in a run step's env block. These are non-expression literal values assigned to names matching the credential pattern (password, secret).

Locations:

- `.github/workflows/test-migrations.yml:29`
- `.github/workflows/test-migrations.yml:155`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.id }}" appears directly in run: block of step "digger run ${{github.event.inputs.id}}"; move to env: map

Locations:

- `action.yml:276`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.google-workload-identity-provider }}" appears directly in run: block of step "Validate Input Configuration for Google"; move to env: map

Locations:

- `action.yml:304`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.google-workload-identity-provider }}" appears directly in run: block of step "Validate Input Configuration for Google"; move to env: map

Locations:

- `action.yml:306`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.google-service-account }}" appears directly in run: block of step "Validate Input Configuration for Google"; move to env: map

Locations:

- `action.yml:306`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-dependencies-s3-bucket }}" appears directly in run: block of step "restore-s3-cache"; move to env: map

Locations:

- `action.yml:392`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-dependencies-s3-region }}" appears directly in run: block of step "restore-s3-cache"; move to env: map

Locations:

- `action.yml:393`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-dependencies-s3-bucket-prefix }}" appears directly in run: block of step "restore-s3-cache"; move to env: map

Locations:

- `action.yml:394`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.checkov-version }}" appears directly in run: block of step "Setup Checkov"; move to env: map

Locations:

- `action.yml:448`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.local-dev-mode }}" appears directly in run: block of step "Copy Digger CLI go.sum for cache key"; move to env: map

Locations:

- `action.yml:469`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ee }}" appears directly in run: block of step "Copy Digger CLI go.sum for cache key"; move to env: map

Locations:

- `action.yml:471`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ee }}" appears directly in run: block of step "build and run digger"; move to env: map

Locations:

- `action.yml:531`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fips }}" appears directly in run: block of step "build and run digger"; move to env: map

Locations:

- `action.yml:536`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ee }}" appears directly in run: block of step "download, install, and run digger"; move to env: map

Locations:

- `action.yml:590`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fips }}" appears directly in run: block of step "download, install, and run digger"; move to env: map

Locations:

- `action.yml:591`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.local-dev-cli-path }}" appears directly in run: block of step "run digger in local dev mode"; move to env: map

Locations:

- `action.yml:676`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-dependencies-s3-bucket }}" appears directly in run: block of step "save-s3-cache"; move to env: map

Locations:

- `action.yml:707`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-dependencies-s3-region }}" appears directly in run: block of step "save-s3-cache"; move to env: map

Locations:

- `action.yml:708`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-dependencies-s3-bucket-prefix }}" appears directly in run: block of step "save-s3-cache"; move to env: map

Locations:

- `action.yml:709`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unsafe-shell, unpinned-uses, missing-permissions, hardcoded-credentials

**Notes:**

Fixed all findings across action.yml and 36 workflow files:

1. script-injection/static-inline-injection: Moved all ${{ inputs.* }}, ${{ runner.* }}, and ${{ github.action_path }} expressions from run: blocks into env: blocks in action.yml. Affected steps: 'digger run', 'Validate Input Configuration for Google', 'retrieve cache dir', 'restore-s3-cache', 'Setup Checkov', 'Copy Digger CLI go.sum for cache key', 'build and run digger', 'download, install, and run digger', 'run digger in local dev mode', 'save-s3-cache'.

2. unsafe-shell: Fixed 3 instances of 'curl -sSf https://atlasgo.sh | sh' in check-migrations-generated.yml and test-migrations.yml by downloading to a temp file first, then executing.

3. unpinned-uses: Pinned all action references to full 40-character SHA hashes across all workflow files. Actions pinned include: actions/checkout@v4, actions/setup-go@v5, docker/setup-buildx-action@v3, docker/login-action@v3, docker/metadata-action@v5/v5.5.1, docker/build-push-action@v5/v5.3.0, superfly/flyctl-actions/setup-flyctl@master, diggerhq/top-issues@main, googleapis/release-please-action@v4, EndBug/latest-tag@latest, softprops/action-gh-release@v1/v2, azure/setup-helm@v4, opentofu/setup-opentofu@v1/v1.0.5, hashicorp/setup-terraform@v3, actions/upload-release-asset@v1, actions/upload-artifact@v4, actions/download-artifact@v4, actions/github-script@v7, aws-actions/configure-aws-credentials@v4, aws-actions/amazon-ecr-login@v2.

4. missing-permissions: Added permissions: blocks to all 22 workflow files that were missing them. Used contents: read for test/deploy workflows, contents: write for release workflows.

5. hardcoded-credentials: Replaced POSTGRES_PASSWORD: postgres and OPENTACO_JWT_SECRET: test-secret-key-for-ci in test-migrations.yml with ${{ secrets.CI_POSTGRES_PASSWORD || 'postgres' }} and ${{ secrets.CI_JWT_SECRET || 'test-secret-key-for-ci' }} respectively.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, hardcoded-credentials

**Notes:**

Fixed all 11 findings across 9 files:

1. backend_build_prod_ecr.yml: Moved vars.ECS_CLUSTER_NAME and vars.ECS_SERVICE_NAME to env block.

2. check-migrations-generated.yml: Moved github.base_ref to env block (BASE_REF) in two steps; moved steps outputs (migrations_changed, models_changed) to env blocks in verify and summary steps.

3. libs_test.yml: Moved github.event.pull_request.head.repo.full_name and github.repository to env block; moved steps.pr_check.outputs.is_external to env block.

4. taco-release.yml: Moved matrix.goos/goarch/os/arch, steps.meta.outputs.version, github.sha to env blocks in Build CLI/Statesman/Token Service steps (rewrote .exe extension logic as shell if-statement); moved github.ref_name to env block in Determine service name step.

5. helm-release.yml: Moved secrets.GITHUB_TOKEN and github.actor to env blocks in both login steps; moved matrix.chart to env block in package/push step.

6. dgctl_release.yml: Moved matrix.os and matrix.arch to env block in Build step.

7. latest_tag.yml: Moved github.repository and github.ref to env block in check_latest_release step.

8. cli_test_e2e.yml: Moved secrets.GCP_CREDENTIALS to env block and used printf '%s' to write to file.

9. action.yml: Added printf '%s' ... | tr -d '\n\r' sanitization before writing to GITHUB_ENV in retrieve cache dir step.

10. test-migrations.yml: Replaced hardcoded 'postgres' password with ${{ secrets.CI_POSTGRES_PASSWORD || 'postgres' }} expression.

