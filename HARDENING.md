<!-- markdownlint-disable -->

# Hardening Report: purcell--setup-emacs/master

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **purcell--setup-emacs/master** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block in action.yml directly interpolates `${{ github.action_path }}` inside the shell command string. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted into the shell command before the shell parses it. Offending line: `run: ${{ github.action_path }}/install.sh`

Locations:

- `action.yml:10`

### github-env-injection (severity: high)

install-nix.sh writes the inherited env var `INPUT_NIX_PATH` (a workflow-controlled value passed in from the calling workflow) directly to `$GITHUB_ENV` without sanitization: `echo "NIX_PATH=${INPUT_NIX_PATH}" >> "$GITHUB_ENV"`. A newline embedded in `INPUT_NIX_PATH` could inject arbitrary environment variables. The required sanitization step (`printf '%s' "$INPUT_NIX_PATH" | tr -d '\n\r'`) is absent. Additionally, `RUNNER_TEMP` (another inherited process env var) is written to `$GITHUB_ENV` without sanitization: `echo "TMPDIR=${RUNNER_TEMP}" >> "$GITHUB_ENV"`.

Locations:

- `install-nix.sh:105`
- `install-nix.sh:110`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

1. action.yml (line 10): Moved `${{ github.action_path }}` from the `run:` shell string into the step's `env:` block as `ACTION_PATH`, and changed the run command to `"$ACTION_PATH/install.sh"` to eliminate the script-injection risk. 2. install-nix.sh (lines 105, 110): Added sanitization for `INPUT_NIX_PATH` and `RUNNER_TEMP` before writing to `$GITHUB_ENV`. Each value is now passed through `printf '%s' "$VAR" | tr -d '\n\r'` to strip embedded newlines/carriage returns before the `echo ... >> "$GITHUB_ENV"` write.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in action.yml line 10: added shell-level double quotes around `$ACTION_PATH/install.sh`. The run: value was changed from `"$ACTION_PATH/install.sh"` (YAML-quoted only) to `"\"$ACTION_PATH/install.sh\""` (YAML-quoted with embedded shell-level double quotes), so the shell now sees `"$ACTION_PATH/install.sh"` and properly quotes the variable expansion.

