<!-- markdownlint-disable -->

# Hardening Report: dfreilich--pack-action/v2.0.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dfreilich--pack-action/v2.0.12** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell commands. (a) action.yml line 32: `pack ${{ inputs.args }}` — the user-controlled `inputs.args` is interpolated directly into the shell command, allowing an attacker to inject arbitrary shell commands via the args input. (a) autorelease.yml line 33: `v=${{ steps.version.outputs.result }}` — a step output (which can be influenced by the workflow_dispatch `tag_name` input) is interpolated directly into the shell. (a) autorelease.yml line 52: `git tag ${{ env.MAJOR_TAG }}` — an env context value is interpolated directly into the shell command.

Locations:

- `action.yml:32`
- `.github/workflows/autorelease.yml:33`
- `.github/workflows/autorelease.yml:52`

### github-env-injection (severity: high)

Unsanitized untrusted data written to $GITHUB_ENV. In autorelease.yml, the 'Bump Release' step interpolates `${{ steps.version.outputs.result }}` (which is derived from the workflow_dispatch `tag_name` input — an attacker-controlled value) into shell variables `bump` and `major`, then writes them to $GITHUB_ENV via `echo "ACTION_TAG=v${bump}" >> $GITHUB_ENV` and `echo "MAJOR_TAG=v${major}" >> $GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline-containing value could inject arbitrary environment variables.

Locations:

- `.github/workflows/autorelease.yml:40`
- `.github/workflows/autorelease.yml:41`

### unpinned-uses (severity: high)

All uses: references and the composite action's uses: step use mutable tag/version refs instead of immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Failing references include: action.yml: `buildpacks/github-actions/setup-pack@v4.1.0`; automerge.yml: `pascalgn/automerge-action@v0.14.1`; autorelease.yml: `actions/checkout@v2`, `actions/github-script@v4.0.2`, `actions/create-release@v1`; main.yml: `actions/checkout@v2`, `docker/login-action@v1`; v1.yml: `actions/checkout@v2`, `dfreilich/pack-action@v1`, `docker/login-action@v1`; v2.yml: `actions/checkout@v2`, `dfreilich/pack-action@v2`, `docker/login-action@v1`.

Locations:

- `action.yml:21`
- `.github/workflows/automerge.yml:14`
- `.github/workflows/autorelease.yml:17`
- `.github/workflows/autorelease.yml:19`
- `.github/workflows/autorelease.yml:44`
- `.github/workflows/main.yml:22`
- `.github/workflows/main.yml:30`
- `.github/workflows/v1.yml:22`
- `.github/workflows/v1.yml:26`
- `.github/workflows/v2.yml:22`
- `.github/workflows/v2.yml:26`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no individual job within any workflow defines a `permissions:` block. This means all jobs run with the default (potentially write-all) token permissions, violating the principle of least privilege. Affected files: automerge.yml, autorelease.yml, main.yml, v1.yml, v2.yml.

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

1. script-injection / static-inline-injection (action.yml): Moved inputs.args to env block as INPUT_ARGS, used xargs-based array tokenization to safely pass arguments to pack.

2. script-injection (autorelease.yml line 33): Moved steps.version.outputs.result to env block as VERSION_RESULT.

3. script-injection (autorelease.yml line 52): Moved env.MAJOR_TAG to step-level env block, used sanitized variable in git commands.

4. github-env-injection (autorelease.yml lines 40-41): Sanitized VERSION_RESULT with tr -d '\n\r' before use, and sanitized ACTION_TAG/MAJOR_TAG values with printf+tr before writing to $GITHUB_ENV.

5. unpinned-uses: Pinned all action references to full 40-char commit SHAs with tag comments: buildpacks/github-actions/setup-pack@b3038dd2, pascalgn/automerge-action@4d2ac8c1, actions/checkout@0717577d, actions/github-script@a3e7071a, actions/create-release@0cb9c9b6, docker/login-action@dd4fa067, dfreilich/pack-action@v1→58beac59, dfreilich/pack-action@v2→a8600da8 (resolved via v2.0.12 tag).

6. missing-permissions: Added permissions blocks to all 5 workflow files with minimal required permissions.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Replaced all 9 occurrences of `${{ env.IMG_NAME }}` inside `run:` shell blocks with `"$IMG_NAME"` across three workflow files: .github/workflows/main.yml (lines 69, 87, 107), .github/workflows/v1.yml (lines 69, 91, 133), and .github/workflows/v2.yml (lines 69, 91, 133). The IMG_NAME variable is already available as a shell environment variable (defined in the workflow-level `env:` block and updated via $GITHUB_ENV in some jobs), so using `"$IMG_NAME"` is both safe and functionally equivalent. Expressions in `with: args:` fields were intentionally left unchanged as they are action inputs, not shell commands.

