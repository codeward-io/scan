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

## 2. Minimal Configuration
Create `.codeward/config.yml`:
```yaml
version: 1
policies:
  vulnerability:
    rules:
      - id: block-critical-new
        action: warn
        filter:
          severity: critical
```

## 3. Open a Pull Request
Add or update a dependency to trigger a diff. The Action posts a summary comment.

## 4. Interpret the Output
- New critical vulnerabilities: flagged under Vulnerabilities table
- No existing backlog noise; only net-new or changed items
- Exit code is non-blocking (`warn`) for first rollout

## 5. Next Steps
- Elevate action to `block` for critical after confidence
- Add license / package / validation policies
- Configure output destinations (JSON, markdown)
- Adopt progressive enforcement (see Progressive Enforcement page)

## What You Get
- Diff-aware focus: only new risk surfaces
- Deterministic JSON arrays for automation
- Simple path to escalated blocking

See also: [Starter Configs](examples/starter-configs.md), [Policy System](concepts/policy-system.md), [Glossary](concepts/glossary.md).
