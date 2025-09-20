# Quick Start

A 5-minute path to seeing Codeward diff-aware policy results on a pull request.

## 1. Add GitHub Action
```yaml
name: Codeward Scan
on: [pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Codeward Scan
        uses: codeward-io/scan-action@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

## 2. (Optional) Configuration
You can run Codeward with **no config file**: an opinionated built-in default is applied (it includes `block` actions for new vulnerability severities (CRITICAL/HIGH/MEDIUM) and certain license severities). If you prefer an initial *non-blocking* / observe-only rollout, add a minimal `config.json` overriding those actions to `warn` (or `info`). See full default: [Auto-Applied Default](configuration/overview.md#default-config).

### Option A: Zero Config (opinionated blocking defaults)
Do nothing. The default will immediately fail the run if new qualifying vulnerabilities or license issues are detected. Use this if you intentionally want strong guardrails from day one.

### Option B: Non-Blocking Minimal Config (override defaults)
Create `.codeward/config.json` (JSON, not YAML) to downgrade initial enforcement:
```json
{
  "vulnerability": [
    {
      "name": "critical-warn",
      "actions": { "new": "warn" },
      "rules": [ { "field": "Severity", "type": "eq", "value": "CRITICAL" } ],
      "outputs": [
        { "format": "markdown", "template": "table", "destination": "git:pr", "fields": ["VulnerabilityID","PkgName","Severity","FixedVersion"], "changes": ["new"] }
      ]
    }
  ]
}
```
Key points:
- JSON schema aligns with the [Main Config Reference](configuration/main-config.md).
- Arrays per domain (`vulnerability`, `license`, `package`, `validation`). Omit domains you are not using yet.
- `actions` keys use canonical change categories: `new, changed, removed, existing`.
- Only `new` is actioned here (non-blocking `warn`) for progressive rollout. Add additional policies later to expand coverage.

### Option C (Optional): Start Progressive Blocking
Add a second policy (e.g. HIGH severities with `warn`) while keeping CRITICAL at `block`—see Starter Configs for patterns.

## 3. Open a Pull Request
Add or update a dependency to trigger a diff. The Action posts a summary comment.

## 4. Interpret the Output
- New critical vulnerabilities: flagged under Vulnerabilities table (or JSON if configured)
- No existing backlog noise in PR view; historical issues appear only if you add outputs targeting `existing` or run baseline scans
- Exit code behavior depends on your chosen option (A may block, B will not)

## 5. Next Steps
- Escalate CRITICAL from `warn` to `block` once stable (if starting with Option B)
- Add license / package / validation policies
- Route existing backlog to a file or issue for remediation tracking
- Configure combined JSON or additional markdown outputs
- Adopt progressive enforcement (see Progressive Enforcement page)

## What You Get
- Diff-aware focus: only new & changed risk surfaces in PRs
- Deterministic JSON arrays for automation
- Flexible starting posture (blocking defaults or non-blocking override) plus baseline/backlog inventory support

See also: [Starter Configs](examples/starter-configs.md), [Policy System](concepts/policy-system.md), [Glossary](concepts/glossary.md), [Main Config Reference](configuration/main-config.md).
