<!-- markdownlint-disable -->

# Hardening Report: purcell--setup-emacs/v8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **purcell--setup-emacs/v8.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a `run:` shell command string. The line `run: ${{ github.action_path }}/install.sh` embeds the `github.action_path` context value directly into the shell command before the shell ever sees it. Even though `github.action_path` is not attacker-controlled in the same way as `github.head_ref`, any `${{ ... }}` expression inside a `run:` block is a script-injection finding per the check rules — the value flows through YAML template substitution before the shell quotes it. The fix is to use the `$GITHUB_ACTION_PATH` environment variable instead: `run: "$GITHUB_ACTION_PATH/install.sh"`.

Locations:

- `action.yml:10`

### github-env-injection (severity: high)

install-nix.sh writes the inherited process environment variable `INPUT_NIX_PATH` to `$GITHUB_ENV` without sanitization: `echo "NIX_PATH=${INPUT_NIX_PATH}" >> "$GITHUB_ENV"`. This is a composite action that inherits env from the calling workflow, so `INPUT_NIX_PATH` is workflow-controlled and must be treated as untrusted. A newline embedded in the value could inject arbitrary variables into the GitHub environment. The fix requires sanitizing the value first: `safe=$(printf '%s' "${INPUT_NIX_PATH}" | tr -d '\n\r')` then `echo "NIX_PATH=${safe}" >> "$GITHUB_ENV"`.

Locations:

- `install-nix.sh:96`

### github-env-injection (severity: high)

install-nix.sh writes the inherited process environment variable `RUNNER_TEMP` to `$GITHUB_ENV` without sanitization: `echo "TMPDIR=${RUNNER_TEMP}" >> "$GITHUB_ENV"`. `RUNNER_TEMP` is set by the calling workflow environment and is therefore workflow-controlled/untrusted for the purposes of this check. A newline embedded in the value could inject arbitrary variables into the GitHub environment. The fix requires sanitizing the value first: `safe=$(printf '%s' "${RUNNER_TEMP}" | tr -d '\n\r')` then `echo "TMPDIR=${safe}" >> "$GITHUB_ENV"`.

Locations:

- `install-nix.sh:101`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three findings: (1) action.yml line 10: replaced `${{ github.action_path }}/install.sh` with `"$GITHUB_ACTION_PATH/install.sh"` to eliminate the template-expression script-injection. (2) install-nix.sh line 96: sanitized INPUT_NIX_PATH with `safe_nix_path=$(printf '%s' "${INPUT_NIX_PATH}" | tr -d '\n\r')` before writing to $GITHUB_ENV. (3) install-nix.sh line 101: sanitized RUNNER_TEMP with `safe_runner_temp=$(printf '%s' "${RUNNER_TEMP}" | tr -d '\n\r')` before writing to $GITHUB_ENV.

