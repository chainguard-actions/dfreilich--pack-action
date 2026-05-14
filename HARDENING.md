# Hardening Report: dfreilich--pack-action/v2.0.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **dfreilich--pack-action/v2.0.13** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run-pack` step in action.yml directly interpolates `${{ inputs.args }}` inside a shell `run:` command (`pack ${{ inputs.args }}`). An attacker who controls the `args` input can inject arbitrary shell commands. The value should be passed via an environment variable and referenced as `$ENV_VAR` in the shell script instead.

Locations:

- `action.yml:33`

### unpinned-uses (severity: high)

The `install` step uses `buildpacks/github-actions/setup-pack@v4.1.0`, which is pinned to a mutable version tag (`v4.1.0`) rather than an immutable 40-character commit SHA. If the tag is moved or the repository is compromised, the action could execute arbitrary code. It should be pinned to a full SHA, e.g. `buildpacks/github-actions/setup-pack@<40-char-sha> # v4.1.0`.

Locations:

- `action.yml:21`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.args }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:34`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

Fixed action.yml: (1) Pinned `buildpacks/github-actions/setup-pack@v4.1.0` to its full commit SHA `b3038dd2ada5d9ce26d9bdd0c4f81473297e4379` with the tag preserved as a comment. (2) Moved `${{ inputs.args }}` from the `run:` shell block into an `env:` block as `PACK_ARGS`, and updated the shell command to reference `$PACK_ARGS` instead, eliminating the script injection vulnerability.

