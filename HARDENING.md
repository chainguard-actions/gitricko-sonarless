<!-- markdownlint-disable -->

# Hardening Report: gitricko--sonarless/v1.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **gitricko--sonarless/v1.4** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: In all three `run:` steps of action.yml, the env var `$ACTION_PATH` (sourced from `${{ github.action_path }}`) is used unquoted in the shell command string. Any `${{ ... }}` value routed through an `env:` block must still be double-quoted when expanded in the shell to prevent word-splitting and glob expansion on metacharacters. Offending lines:
- Step 'Get Docker Deps' (line 31): `$ACTION_PATH/makefile.sh docker-deps-get` and (line 32): `$ACTION_PATH/makefile.sh sonar-ext-get`
- Step 'Scanning' (line 44): `run: $ACTION_PATH/makefile.sh scan`
- Step 'Scan Results' (line 56): `run: $ACTION_PATH/makefile.sh results`
All three should use `"$ACTION_PATH/makefile.sh"` with double quotes.

Locations:

- `action.yml:31`
- `action.yml:32`
- `action.yml:44`
- `action.yml:56`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all four unquoted $ACTION_PATH usages in action.yml. In all three run: steps ('Get Docker Deps', 'Scanning', 'Scan Results'), the shell invocations of $ACTION_PATH/makefile.sh were updated to use double quotes: "$ACTION_PATH/makefile.sh". This prevents word-splitting and glob expansion when the action path contains spaces or shell metacharacters.

