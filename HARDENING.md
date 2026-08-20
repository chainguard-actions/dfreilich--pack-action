<!-- markdownlint-disable -->

# Hardening Report: dfreilich--pack-action/v2.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dfreilich--pack-action/v2.1.1** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ inputs.args }}` is directly interpolated into a `run:` shell command in action.yml. Since `inputs.args` is caller-controlled, an attacker can inject arbitrary shell commands. The offending line is: `pack ${{ inputs.args }}`

Locations:

- `action.yml:33`

### script-injection (severity: high)

Sub-rule (a): `${{ steps.version.outputs.result }}` is directly interpolated into a `run:` shell command in autorelease.yml. Step outputs are workflow-controllable and flow through YAML template substitution before the shell sees them. The offending line is: `v=${{ steps.version.outputs.result }}`

Locations:

- `.github/workflows/autorelease.yml:33`

### github-env-injection (severity: high)

In autorelease.yml, the `Bump Release` step assigns `${{ steps.version.outputs.result }}` (an untrusted step output) to the shell variable `v`, then writes derived values (`v${bump}`, `v${major}`) to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline injected into the step output could poison subsequent environment variables read from GITHUB_ENV.

Locations:

- `.github/workflows/autorelease.yml:33`

### unpinned-uses (severity: high)

Multiple files reference GitHub Actions and reusable actions using mutable tag refs instead of pinned 40-character SHA commit digests, making them vulnerable to supply-chain attacks if the tag is moved. Failing references include:
- action.yml: `buildpacks/github-actions/setup-pack@v4.6.0`
- automerge.yml: `pascalgn/automerge-action@v0.14.3`
- autorelease.yml: `actions/checkout@v2`, `actions/github-script@v5`, `actions/create-release@v1`
- main.yml: `actions/checkout@v3`, `docker/login-action@v1`
- v1.yml: `actions/checkout@v2`, `docker/login-action@v1`, `dfreilich/pack-action@v1`
- v2.yml: `actions/checkout@v2`, `docker/login-action@v1`, `dfreilich/pack-action@v2`

Locations:

- `action.yml:20`
- `.github/workflows/automerge.yml:14`
- `.github/workflows/autorelease.yml:14`
- `.github/workflows/main.yml:20`
- `.github/workflows/v1.yml:18`
- `.github/workflows/v2.yml:18`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no individual job within any workflow defines a `permissions:` block. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

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

Fixed all findings across action.yml and all workflow files:

1. **script-injection / static-inline-injection (action.yml)**: Moved `${{ inputs.args }}` out of the `run:` block into an `env:` block as `INPUT_ARGS`. Used xargs-based array tokenization (`while IFS= read -r -d '' t; do opts+=("$t"); done < <(printf '%s' "$INPUT_ARGS" | xargs printf '%s\0')`) to safely split the args list and pass it to `pack`.

2. **script-injection + github-env-injection (autorelease.yml)**: Moved `${{ steps.version.outputs.result }}` to an `env:` block as `VERSION_RESULT`. Sanitized the value with `printf '%s' "$VERSION_RESULT" | tr -d '\n\r'` before use, and also sanitized derived values (`safe_bump`, `safe_major`) before writing to `$GITHUB_ENV`. Fixed `git tag` to use the env var safely.

3. **unpinned-uses**: Pinned all actions to full 40-char SHA digests:
   - `buildpacks/github-actions/setup-pack@v4.6.0` → `@918407dc3eb8c209c5b69902b5024ebcb63fe3b5`
   - `pascalgn/automerge-action@v0.14.3` → `@04dfc9eae2586d19b7362d4f6413c48135d9c25a`
   - `actions/checkout@v2` → `@0717577d45739eb3c851188b29f50ed6c0b2194e`
   - `actions/checkout@v3` → `@a37ce9120846195fa4ece8f58b268e6043cb2f26`
   - `actions/github-script@v5` → `@211cb3fefb35a799baa5156f9321bb774fe56294`
   - `actions/create-release@v1` → `@0cb9c9b65d5d1901c1f53e5e66eaf4afd303e70e`
   - `docker/login-action@v1` → `@dd4fa0671be5250ee6f50aedf4cb05514abda2c7`
   - `dfreilich/pack-action@v1` → `@58beac59ac4bdfd6b8d61779c3446ee4e3d107a7`
   - `dfreilich/pack-action@v2` → `@6e760443286e2a7f651646364df366a553dcff71` (v2.1.1, the latest v2 tag found)

4. **missing-permissions**: Added `permissions:` blocks to all workflow files:
   - `automerge.yml`: `contents: write, pull-requests: write` (needed for automerge)
   - `autorelease.yml`: `contents: write` (needed to create releases and push tags)
   - `main.yml`, `v1.yml`, `v2.yml`: `contents: read` (minimal for checkout/testing)

