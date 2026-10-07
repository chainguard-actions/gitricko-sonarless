<!-- markdownlint-disable -->

# Hardening Report: gitricko--sonarless/v1.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gitricko--sonarless/v1.4** was hardened automatically. 1 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

All three `run:` blocks in action.yml use `$ACTION_PATH` unquoted in shell commands. `ACTION_PATH` is set from `${{ github.action_path }}` — per rule (b), any env var holding a `${{ ... }}` expression must be double-quoted when expanded in a `run:` script to prevent shell metacharacter injection. Additionally, the 'Scanning' and 'Scan Results' steps set `SONAR_PROJECT_NAME`, `SONAR_PROJECT_KEY`, `SONAR_SOURCE_PATH`, `SONAR_METRICS_PATH`, and `SONAR_INSTANCE_PORT` from `inputs.*` (attacker-controlled), and these are passed unquoted to `makefile.sh` where they are interpolated unquoted into curl URL query strings (e.g., `?name=${SONAR_PROJECT_NAME}&project=${SONAR_PROJECT_NAME}`) and Docker `-e` flags (e.g., `SONAR_SCANNER_OPTS="-Dsonar.projectKey=${SONAR_PROJECT_NAME} -Dsonar.sources=${SONAR_SOURCE_PATH}"`), allowing shell metacharacter injection. Offending lines: `$ACTION_PATH/makefile.sh docker-deps-get` (line 30), `$ACTION_PATH/makefile.sh sonar-ext-get` (line 31), `run: $ACTION_PATH/makefile.sh scan` (line 44), `run: $ACTION_PATH/makefile.sh results` (line 57).

Locations:

- `action.yml:30`
- `action.yml:31`
- `action.yml:44`
- `action.yml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all script-injection findings:
1. action.yml: Quoted all four $ACTION_PATH usages as "$ACTION_PATH/makefile.sh" to prevent shell metacharacter injection from the github.action_path expression (lines 30, 31, 44, 57).
2. makefile.sh scan(): Replaced inline URL query string interpolation with --data-urlencode for SONAR_PROJECT_NAME in curl calls (projects/create, users/set_homepage, qualitygates/project_status). Refactored docker run to export SONAR_SCANNER_OPTS as a separate env var and pass it via --env SONAR_SCANNER_OPTS (no embedded value in the -e flag), preventing injection via SONAR_PROJECT_NAME or SONAR_SOURCE_PATH.
3. makefile.sh results(): Replaced inline URL query string interpolation with -G --data-urlencode for SONAR_PROJECT_NAME in the measures/component curl call.

### Iteration 2

**Fixes applied:** invalid-yaml

**Notes:**

Fixed two invalid YAML `run:` values in action.yml. Both `run: "$ACTION_PATH/makefile.sh" scan` (line 34) and `run: "$ACTION_PATH/makefile.sh" results` (line 47) were single-line run values starting with a double-quoted string, which YAML parsed as a complete scalar and then rejected the trailing argument word. Converted both to block scalar form (`run: |`) so the full command is treated as literal text and the YAML parses correctly.

### Iteration 3

**Fixes applied:** hardcoded-credentials

**Notes:**

Removed all hardcoded literal credentials from makefile.sh. Added SONAR_ADMIN_INITIAL_PASSWORD env var (defaults to the well-known SonarQube factory default 'admin') and SONAR_ADMIN_PASSWORD env var (generated randomly at runtime via /dev/urandom if not provided externally). All 6 curl commands that previously embedded 'admin:admin' or 'admin:Son@rless123' now use ${SONAR_ADMIN_INITIAL_PASSWORD} and ${SONAR_ADMIN_PASSWORD} references. The echo messages that previously printed the literal password now show a placeholder string instead.

