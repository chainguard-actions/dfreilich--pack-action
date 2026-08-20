<!-- markdownlint-disable -->

# Hardening Report: dfreilich--pack-action/v2.0.15

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dfreilich--pack-action/v2.0.15** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct expression interpolation in a run: block. In action.yml, `pack ${{ inputs.args }}` interpolates the user-controlled `inputs.args` directly into the shell command, allowing an attacker to inject arbitrary shell commands by supplying crafted arguments (e.g. containing semicolons, backticks, or subshell syntax). In autorelease.yml, `v=${{ steps.version.outputs.result }}` interpolates a step output directly into the shell, which can be attacker-influenced via workflow_dispatch inputs.

Locations:

- `action.yml:28`
- `.github/workflows/autorelease.yml:27`

### github-env-injection (severity: high)

In autorelease.yml, the `Bump Release` step writes values derived from `steps.version.outputs.result` (an untrusted/workflow-controllable step output) into $GITHUB_ENV via `echo "ACTION_TAG=v${bump}" >> $GITHUB_ENV` and `echo "MAJOR_TAG=v${major}" >> $GITHUB_ENV`. The values `${bump}` and `${major}` are computed from `v=${{ steps.version.outputs.result }}` without any sanitization (no `printf '%s' ... | tr -d '\n\r'` step), allowing newline injection that could set arbitrary environment variables for subsequent steps.

Locations:

- `.github/workflows/autorelease.yml:32`
- `.github/workflows/autorelease.yml:33`

### unpinned-uses (severity: high)

Every `uses:` reference across action.yml and all workflow files uses a mutable tag or version string instead of a pinned 40-character commit SHA. This exposes the action to supply-chain attacks if any upstream action or reusable workflow is compromised or its tag is moved. Failing references include: action.yml: `buildpacks/github-actions/setup-pack@v4.1.0`; automerge.yml: `pascalgn/automerge-action@v0.14.3`; autorelease.yml: `actions/checkout@v2`, `actions/github-script@v5`, `actions/create-release@v1`; main.yml: `actions/checkout@v2`, `docker/login-action@v1`; v1.yml: `actions/checkout@v2`, `dfreilich/pack-action@v1`, `docker/login-action@v1`; v2.yml: `actions/checkout@v2`, `dfreilich/pack-action@v2`, `docker/login-action@v1`.

Locations:

- `action.yml:18`
- `.github/workflows/automerge.yml:14`
- `.github/workflows/autorelease.yml:14`
- `.github/workflows/autorelease.yml:18`
- `.github/workflows/autorelease.yml:36`
- `.github/workflows/main.yml:14`
- `.github/workflows/main.yml:27`
- `.github/workflows/v1.yml:22`
- `.github/workflows/v1.yml:30`
- `.github/workflows/v2.yml:14`
- `.github/workflows/v2.yml:22`

### missing-permissions (severity: medium)

None of the five workflow files (automerge.yml, autorelease.yml, main.yml, v1.yml, v2.yml) define a top-level `permissions:` key, and no individual job within any of these files defines a `permissions:` key either. Without explicit permissions, workflows run with the default (often broad) token permissions, violating the principle of least privilege. This is especially concerning for automerge.yml (which uses pull_request_target-adjacent triggers) and autorelease.yml (which creates releases and pushes tags).

Locations:

- `.github/workflows/automerge.yml:1`
- `.github/workflows/autorelease.yml:1`
- `.github/workflows/main.yml:1`
- `.github/workflows/v1.yml:1`
- `.github/workflows/v2.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.args }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:34`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all 5 findings across action.yml and 5 workflow files:

1. **script-injection / static-inline-injection** (action.yml): Moved `inputs.args` to env block as `INPUT_ARGS`, then used xargs-based tokenization into a bash array before passing to `pack`. This prevents shell injection while correctly handling quoted arguments.

2. **script-injection** (autorelease.yml): Moved `steps.version.outputs.result` to env block as `VERSION_RESULT`, then sanitized with `printf '%s' | tr -d '\n\r'` before use.

3. **github-env-injection** (autorelease.yml): Added `printf '%s' | tr -d '\n\r'` sanitization for both `bump` and `major` values before writing to `$GITHUB_ENV`. Also quoted `$GITHUB_ENV` and `$MAJOR_TAG` references.

4. **unpinned-uses**: Pinned all action references to full 40-char commit SHAs:
   - `buildpacks/github-actions/setup-pack@b3038dd2ada5d9ce26d9bdd0c4f81473297e4379` (v4.1.0)
   - `pascalgn/automerge-action@04dfc9eae2586d19b7362d4f6413c48135d9c25a` (v0.14.3)
   - `actions/checkout@0717577d45739eb3c851188b29f50ed6c0b2194e` (v2)
   - `actions/github-script@211cb3fefb35a799baa5156f9321bb774fe56294` (v5)
   - `actions/create-release@0cb9c9b65d5d1901c1f53e5e66eaf4afd303e70e` (v1)
   - `docker/login-action@dd4fa0671be5250ee6f50aedf4cb05514abda2c7` (v1)
   - `dfreilich/pack-action@58beac59ac4bdfd6b8d61779c3446ee4e3d107a7` (v1)
   - `dfreilich/pack-action@8e1256f08f082c81266d4e7cb9ea2fe6a38b8884` (v2.0.15, since v2 tag doesn't exist)

5. **missing-permissions**: Added `permissions:` blocks to all 5 workflow files with minimal required permissions (automerge.yml: pull-requests+contents write; autorelease.yml: contents write; main/v1/v2.yml: contents read).

