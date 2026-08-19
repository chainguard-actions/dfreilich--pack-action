<!-- markdownlint-disable -->

# Hardening Report: dfreilich--pack-action/v2.0.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dfreilich--pack-action/v2.0.13** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): The `run-pack` step in action.yml directly interpolates `${{ inputs.args }}` inside a shell `run:` block (`pack ${{ inputs.args }}`). An attacker who controls the `args` input can inject arbitrary shell commands. The expression is substituted by the YAML template engine before the shell ever sees it, bypassing any quoting.

Locations:

- `action.yml:30`

### script-injection (severity: high)

Rule (a): The `Bump Release` step in autorelease.yml directly interpolates `${{ steps.version.outputs.result }}` inside a `run:` block (`v=${{ steps.version.outputs.result }}`). The value originates from a `workflow_dispatch` input (`tag_name`) which is attacker-controllable, allowing shell command injection. Additionally, the `Update major tag` step interpolates `${{ env.MAJOR_TAG }}` (derived from the same tainted source) directly in `run:` commands (`git tag ${{ env.MAJOR_TAG }}` and `git push -f origin ${{ env.MAJOR_TAG }}`).

Locations:

- `.github/workflows/autorelease.yml:33`
- `.github/workflows/autorelease.yml:51`
- `.github/workflows/autorelease.yml:52`

### github-env-injection (severity: high)

The `Bump Release` step in autorelease.yml writes values derived from `steps.version.outputs.result` (which is sourced from the attacker-controllable `workflow_dispatch` input `tag_name`) to `$GITHUB_ENV` without sanitization (`echo "ACTION_TAG=v${bump}" >> $GITHUB_ENV` and `echo "MAJOR_TAG=v${major}" >> $GITHUB_ENV`). A newline character in the input could inject additional environment variable definitions, leading to environment poisoning for subsequent steps. The required sanitization step (`printf '%s' ... | tr -d '\n\r'`) is absent.

Locations:

- `.github/workflows/autorelease.yml:37`
- `.github/workflows/autorelease.yml:38`

### unpinned-uses (severity: high)

action.yml references `buildpacks/github-actions/setup-pack@v4.1.0` — a mutable tag ref, not a pinned 40-character SHA commit hash. If the tag is moved or the repository is compromised, the action will silently execute different code.

Locations:

- `action.yml:21`

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files use mutable tag or branch refs instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks: `actions/checkout@v2`, `pascalgn/automerge-action@v0.14.1`, `actions/github-script@v4.0.2`, `actions/create-release@v1`, `docker/login-action@v1`, `dfreilich/pack-action@v1`, `dfreilich/pack-action@v2`.

Locations:

- `.github/workflows/automerge.yml:14`
- `.github/workflows/autorelease.yml:17`
- `.github/workflows/autorelease.yml:19`
- `.github/workflows/autorelease.yml:40`
- `.github/workflows/main.yml:21`
- `.github/workflows/main.yml:27`
- `.github/workflows/main.yml:52`
- `.github/workflows/main.yml:73`
- `.github/workflows/main.yml:88`
- `.github/workflows/v1.yml:23`
- `.github/workflows/v1.yml:29`
- `.github/workflows/v1.yml:47`
- `.github/workflows/v1.yml:60`
- `.github/workflows/v1.yml:68`
- `.github/workflows/v2.yml:22`
- `.github/workflows/v2.yml:28`
- `.github/workflows/v2.yml:46`
- `.github/workflows/v2.yml:59`
- `.github/workflows/v2.yml:67`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and no individual job within them defines a `permissions:` key either. Without explicit permissions, workflows run with the default token permissions (which may be `write-all` depending on repository settings), granting unnecessarily broad access to the GITHUB_TOKEN.

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

Fixed all 7 findings across action.yml and 5 workflow files:

1. script-injection (action.yml): Moved inputs.args from run: block to env: block as INPUT_ARGS, then used xargs-based array tokenization to safely pass args to pack.

2. script-injection (autorelease.yml): Moved steps.version.outputs.result to env: block as VERSION_RESULT; moved env.MAJOR_TAG to env: block in 'Update major tag' step; all run: commands use plain $VAR references.

3. github-env-injection (autorelease.yml): Added printf '%s' ... | tr -d '\n\r' sanitization for all values written to $GITHUB_ENV.

4. unpinned-uses (action.yml): Pinned buildpacks/github-actions/setup-pack@v4.1.0 to SHA b3038dd2ada5d9ce26d9bdd0c4f81473297e4379.

5. unpinned-uses (workflow files): Pinned all mutable tag refs in automerge.yml, autorelease.yml, main.yml, v1.yml, v2.yml to full 40-char SHAs.

6. missing-permissions: Added top-level permissions blocks to all 5 workflow files with minimal required permissions.

7. static-inline-injection: Same fix as script-injection for action.yml.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 9 script-injection occurrences and 6 github-env-injection occurrences across main.yml, v1.yml, and v2.yml. For script-injection: replaced `${{ env.IMG_NAME }}` with `"$IMG_NAME"` (shell variable) in all `run:` blocks in dockerhub_remote_build, github_registry_remote_build, and local_with_secure_builder Test App steps. For github-env-injection: rewrote all 'Set App Name' steps to sanitize USERNAME, IMG_NAME, and REGISTRY values using `printf '%s' "${VAR}" | tr -d '\n\r'` before writing to $GITHUB_ENV. The remaining ${{ env.IMG_NAME }} expressions in `with: args:` blocks are action inputs (not shell commands) and are not affected by the script-injection rule.

