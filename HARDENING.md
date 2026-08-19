<!-- markdownlint-disable -->

# Hardening Report: game-ci--unity-activate/v3.0.0-beta.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **game-ci--unity-activate/v3.0.0-beta.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

The `UNITY_LICENSE` environment variable in `.github/workflows/main.yml` is assigned a literal Unity license XML blob (containing machine bindings, a serial hash, developer data, and a cryptographic signature) directly in the workflow file. This is a hardcoded credential — it should be stored as a GitHub Actions secret and referenced via `${{ secrets.UNITY_LICENSE }}` instead.

Locations:

- `.github/workflows/main.yml:7`

### script-injection (severity: high)

Sub-rule (a): The `run:` block in the `requestActivation` job directly interpolates `${{ matrix.unityVersion }}` inside a shell command string: `echo "m_EditorVersion: ${{ matrix.unityVersion }}" > ProjectSettings/ProjectVersion.txt`. The `matrix.*` context flows through YAML template substitution before the shell processes it, allowing an attacker who can influence matrix values to inject arbitrary shell commands. The value should be passed via an `env:` variable and double-quoted in the shell script instead.

Locations:

- `.github/workflows/main.yml:93`

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable tags rather than immutable 40-character SHA digests, making them vulnerable to supply-chain attacks if the referenced tag is moved or overwritten. Failing references:
- `actions/checkout@v4` (appears twice in main.yml)
- `actions/setup-node@v4` (main.yml)
- `actions/cache@v4` (main.yml)
- `Actions-R-Us/actions-tagger@v2` (versioning.yml)
All should be pinned to full commit SHAs, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/main.yml:18`
- `.github/workflows/main.yml:28`
- `.github/workflows/main.yml:38`
- `.github/workflows/main.yml:72`
- `.github/workflows/versioning.yml:11`

### missing-permissions (severity: medium)

Neither `.github/workflows/main.yml` nor `.github/workflows/versioning.yml` declares a top-level `permissions:` key, and no individual job within either file declares its own `permissions:` block. Without explicit permissions, workflows run with the repository's default token permissions (which may be `write-all` for older repositories), granting unnecessarily broad access. A minimal `permissions:` block (e.g. `permissions: read-all` or specific scopes) should be added at the top level or per-job.

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/versioning.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** hardcoded-credentials, script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings in .github/workflows/main.yml and .github/workflows/versioning.yml:
1. hardcoded-credentials: Replaced literal Unity license XML blob with ${{ secrets.UNITY_LICENSE }} reference.
2. script-injection: Moved ${{ matrix.unityVersion }} into an env: block (UNITY_VERSION) and referenced it as $UNITY_VERSION in the shell script.
3. unpinned-uses: Pinned actions/checkout@v4 (SHA: 11d5960a...), actions/setup-node@v4 (SHA: 49933ea5...), actions/cache@v4 (SHA: 0057852b...), and Actions-R-Us/actions-tagger@v2 (SHA: 330ddfac...) to full commit SHAs with tag comments.
4. missing-permissions: Added 'permissions: contents: read' to main.yml and 'permissions: contents: write' to versioning.yml (write needed for tag management).

