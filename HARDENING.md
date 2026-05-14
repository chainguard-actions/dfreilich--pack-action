# Hardening Report: dfreilich--pack-action/v2.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **dfreilich--pack-action/v2.1.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run-pack` step directly interpolates `${{ inputs.args }}` inside a `run:` shell command: `pack ${{ inputs.args }}`. An attacker who controls the `args` input can inject arbitrary shell commands. The value must be assigned to an environment variable via `env:` and referenced as `$ENV_VAR` in the shell command instead.

Locations:

- `action.yml:30`

### unpinned-uses (severity: high)

The `uses:` reference `buildpacks/github-actions/setup-pack@v4.6.0` uses a mutable version tag (`@v4.6.0`) rather than a pinned 40-character commit SHA. If the tag is moved or the upstream repository is compromised, the action will silently execute different code. Pin to a full SHA, e.g. `buildpacks/github-actions/setup-pack@<40-char-sha> # v4.6.0`.

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

Fixed three findings in action.yml: (1) Pinned `buildpacks/github-actions/setup-pack@v4.6.0` to its full commit SHA `918407dc3eb8c209c5b69902b5024ebcb63fe3b5` with the tag preserved as a comment. (2) & (3) Moved `${{ inputs.args }}` out of the `run:` shell block into an `env:` block as `PACK_ARGS`, and referenced it as `$PACK_ARGS` in the shell command to prevent shell injection.

