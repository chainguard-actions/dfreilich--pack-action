# Hardening Report: dfreilich--pack-action/v2.0.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **dfreilich--pack-action/v2.0.12** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run-pack` step directly interpolates the attacker-controlled expression `${{ inputs.args }}` into a shell `run:` command: `pack ${{ inputs.args }}`. An attacker who controls the `args` input can inject arbitrary shell commands (e.g. by passing `; malicious-command`). The value must be assigned to an environment variable via `env:` and referenced as `$ENV_VAR` in the shell command instead.

Locations:

- `action.yml:33`

### unpinned-uses (severity: high)

The `install` step references `buildpacks/github-actions/setup-pack@v4.1.0`, which is pinned to a mutable version tag (`v4.1.0`) rather than an immutable 40-character commit SHA. If the tag is moved or the upstream repository is compromised, the action will silently execute different code. Pin to a full SHA digest, e.g. `buildpacks/github-actions/setup-pack@<40-char-sha> # v4.1.0`.

Locations:

- `action.yml:20`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.args }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:34`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

Fixed two categories of issues in action.yml: (1) Pinned `buildpacks/github-actions/setup-pack@v4.1.0` to its full immutable commit SHA `b3038dd2ada5d9ce26d9bdd0c4f81473297e4379` with the tag preserved as a comment. (2) Moved the attacker-controlled `${{ inputs.args }}` expression out of the `run:` shell block into an `env:` map as `PACK_ARGS`, and referenced it as `$PACK_ARGS` in the shell command to prevent shell injection.

