# Hardening Report: dfreilich--pack-action/v2.0.15

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **dfreilich--pack-action/v2.0.15** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run-pack` step directly interpolates `${{ inputs.args }}` into a shell command: `pack ${{ inputs.args }}`. An attacker who controls the `args` input can inject arbitrary shell commands. The value must be assigned to an environment variable first and referenced as `$ENV_VAR` in the shell command.

Locations:

- `action.yml:28`

### unpinned-uses (severity: high)

The `uses:` reference `buildpacks/github-actions/setup-pack@v4.1.0` uses a mutable version tag (`@v4.1.0`) rather than a full 40-character commit SHA. A compromised or altered tag could introduce malicious code. Pin to a specific commit SHA, e.g. `buildpacks/github-actions/setup-pack@<40-char-sha> # v4.1.0`.

Locations:

- `action.yml:18`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.args }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:34`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

1. Pinned `buildpacks/github-actions/setup-pack@v4.1.0` to full SHA `b3038dd2ada5d9ce26d9bdd0c4f81473297e4379` with the tag preserved as a comment. 2. Fixed script-injection/static-inline-injection by moving `${{ inputs.args }}` out of the `run:` block into an `env:` map as `PACK_ARGS`, then referencing it as `$PACK_ARGS` in the shell command — preventing attacker-controlled input from being interpreted as shell commands.

