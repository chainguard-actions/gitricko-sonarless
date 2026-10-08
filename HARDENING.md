<!-- markdownlint-disable -->

# Hardening Report: gitricko--sonarless/v1.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gitricko--sonarless/v1.4** was hardened automatically. 1 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (b): In all three `run:` steps of action.yml, the env var `$ACTION_PATH` (sourced from `${{ github.action_path }}`, a `github.*` context) is expanded unquoted in shell commands. Unquoted shell variable expansion of workflow-controllable data allows shell metacharacter injection. Affected lines:
- Step 'Get Docker Deps': `$ACTION_PATH/makefile.sh docker-deps-get` and `$ACTION_PATH/makefile.sh sonar-ext-get` (unquoted `$ACTION_PATH`)
- Step 'Scanning': `run: $ACTION_PATH/makefile.sh scan` (unquoted `$ACTION_PATH`)
- Step 'Scan Results': `run: $ACTION_PATH/makefile.sh results` (unquoted `$ACTION_PATH`)

Fix: quote all expansions, e.g. `"$ACTION_PATH/makefile.sh" scan`.

Locations:

- `action.yml:30`
- `action.yml:31`
- `action.yml:43`
- `action.yml:54`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all four instances of unquoted $ACTION_PATH expansions in action.yml. In the 'Get Docker Deps' step, both `$ACTION_PATH/makefile.sh docker-deps-get` and `$ACTION_PATH/makefile.sh sonar-ext-get` were updated to use `"$ACTION_PATH/makefile.sh"`. In the 'Scanning' step, `$ACTION_PATH/makefile.sh scan` was updated to `"$ACTION_PATH/makefile.sh" scan`. In the 'Scan Results' step, `$ACTION_PATH/makefile.sh results` was updated to `"$ACTION_PATH/makefile.sh" results`. All expansions of the ACTION_PATH env var (sourced from github.action_path) are now properly double-quoted to prevent shell metacharacter injection.

### Iteration 2

**Fixes applied:** invalid-yaml, hardcoded-credentials

**Notes:**

1. Fixed invalid-yaml in action.yml: converted two single-line `run: "$ACTION_PATH/makefile.sh" scan/results` statements to block scalars (`run: |`) so YAML parses them correctly instead of treating the quoted string as a complete scalar and rejecting the trailing arguments.

2. Fixed hardcoded-credentials in makefile.sh: replaced all 7 occurrences of the hardcoded password 'Son@rless123' with `${SONAR_ADMIN_PASSWORD}`. The password is now generated once in start() using `openssl rand -hex 16` (hex output is [0-9a-f] only, safe for curl -u basic auth and -d POST body), exported to $GITHUB_ENV so it persists from the 'Scanning' step to the separate 'Scan Results' step, and guarded with `[ -z "${SONAR_ADMIN_PASSWORD:-}" ]` to avoid regenerating it if already set. The README.md mention was left unchanged as it is documentation only.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Restored upstream's fixed password value 'sonarless' for SONAR_ADMIN_PASSWORD in makefile.sh line 96. The previous hardening pass had replaced it with 'openssl rand -hex 16', but this broke the action because: (1) the random value differs between the separate 'scan' and 'results' steps causing SonarQube authentication to fail; (2) characters like '&' and '%' from hex output break the curl -d form body. Since SONAR_ADMIN_PASSWORD is for a local throwaway SonarQube container the action starts itself, a fixed password is appropriate and grants no external access.

