---
title: GitHub Actions Integration
description: Integrate Codeward into GitHub pull requests and main branch workflows for diff-aware policy gating with minimal setup.
---

# GitHub Actions Integration

Integrate **Codeward** into GitHub pull request and main branch workflows to govern code (human or AI‑generated) before merge. The published composite action `codeward-io/scan` wraps the container and performs event‑aware diff scanning automatically.

## Why Use the GitHub Action
- No custom scripting for diff logic (base + head handled internally)
- Deterministic, policy‑driven gating (`info | warn | block`)
- Fast feedback on only what changed (noise reduction for reviews)
- Enforces vulnerability, license, package, and validation policies early

## Quick Start (Minimal)
Create `.github/workflows/codeward-scan.yml`:
```yaml
name: Codeward
on:
  pull_request:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read          # checkout (performed inside action)
      packages: read          # pull GHCR image
      pull-requests: write    # comment on PRs
      issues: write           # create/update issues on main scans (optional)
    steps:
      - uses: codeward-io/scan@v0.0.1
        with:
          event: ${{ github.event_name }}
          repository: ${{ github.repository }}
          current_branch: ${{ github.ref }}
          pr_number: ${{ github.event.number }}
          token: ${{ github.token }}
```
PR events perform a diff (base vs head). Non‑PR events scan the default branch checkout.

## Permissions Rationale
| Permission | Why Needed | Scope Reduction Tip |
|------------|-----------|---------------------|
| contents: read | Access repository checkout (internally) | Default minimal read only |
| packages: read | Pull scanner image from GHCR | None (required) |
| pull-requests: write | Post/update PR comment | Remove if not using `git:pr` destinations |
| issues: write | Create/update issue for backlog (`existing`) | Remove if not using `git:issue` |

> Use the least set matching configured destinations. Removing a permission simply skips that destination.

## Improving Accuracy with Dependency Installation
Installing dependencies enriches vulnerability & license detection (transitive graph). Two models:
1. Action only (fastest; may miss some transitive context without installs)
2. Explicit checkouts + dependency installs + direct container run (maximum fidelity)

<details>
<summary>Advanced Workflow (PR with Dependency Installs)</summary>

```yaml
name: Codeward (PR with deps)
on: pull_request

jobs:
  scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: read
      pull-requests: write
      issues: write
    steps:
      # Explicit checkouts (base + head)
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.base_ref }}
          path: main
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.head_ref }}
          path: branch

      # Example: Node.js dependency resolution for both trees
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: |
            main/package-lock.json
            branch/package-lock.json
      - name: Install deps (base)
        if: hashFiles('main/package-lock.json') != ''
        run: npm ci
        working-directory: main
      - name: Install deps (head)
        if: hashFiles('branch/package-lock.json') != ''
        run: npm ci
        working-directory: branch

      - name: Pull scanner image
        run: |
          echo ${{ secrets.GITHUB_TOKEN }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin
          docker pull ghcr.io/codeward-io/scan:v0.0.1

      - name: Run Codeward diff scan
        run: |
          mkdir -p results cache
          docker run --rm --name codeward-scan \
            -v ${PWD}/main:/main:rw \
            -v ${PWD}/branch:/branch:rw \
            -v ${PWD}/results:/results:rw \
            -v ${PWD}/cache:/.cache:rw \
            -e GITHUB_TOKEN=${{ secrets.GITHUB_TOKEN }} \
            -e GITHUB_OWNER=${{ github.repository_owner }} \
            -e GITHUB_REPO=${{ github.event.repository.name }} \
            -e GITHUB_PR_NR=${{ github.event.number }} \
            -e CI_EVENT="pr" \
            -e CI=true \
            ghcr.io/codeward-io/scan:v0.0.1
```
</details>

### Push (Main) Example (Single Checkout)
```yaml
name: Codeward (main push)
on:
  push:
    branches: [main]

jobs:
  scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: read
      issues: write
    steps:
      - uses: actions/checkout@v4
        with:
          path: main

      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: main/package-lock.json
      - name: Install deps
        if: hashFiles('main/package-lock.json') != ''
        run: npm ci
        working-directory: main

      - name: Pull scanner image
        run: |
          echo ${{ secrets.GITHUB_TOKEN }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin
          docker pull ghcr.io/codeward-io/scan:v0.0.1

      - name: Run Codeward scan (main)
        run: |
          mkdir -p results cache
          docker run --rm --name codeward-scan \
            -v ${PWD}/main:/main:rw \
            -v ${PWD}/results:/results:rw \
            -v ${PWD}/cache:/.cache:rw \
            -e GITHUB_TOKEN=${{ secrets.GITHUB_TOKEN }} \
            -e GITHUB_OWNER=${{ github.repository_owner }} \
            -e GITHUB_REPO=${{ github.event.repository.name }} \
            -e CI_EVENT="main" \
            -e CI=true \
            ghcr.io/codeward-io/scan:v0.0.1
```

## Optional Conditional Execution
Skip scans on non‑impacting changes via a path filter action (e.g., `dorny/paths-filter`). Only run Codeward when source or dependency files change.

## Example Configuration (Git Destinations)
```json
{
  "vulnerability": [
    {
      "name": "critical-and-high-vulns",
      "actions": {"new": "block", "changed": "warn", "removed": "info", "existing": "warn"},
      "rules": [
        {"field": "Severity", "type": "eq", "value": "CRITICAL"},
        {"field": "Severity", "type": "eq", "value": "HIGH"}
      ],
      "outputs": [
        {
          "format": "markdown",
          "template": "table",
          "destination": "git:pr",
          "title": "🔒 Critical / High Vulnerabilities (New & Changed)",
          "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
          "changes": ["new", "changed"],
          "collapse": true
        },
        {
          "format": "markdown",
          "template": "text",
          "destination": "git:issue",
          "title": "🚨 Existing Vulnerabilities (Backlog)",
          "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
          "changes": ["existing"],
          "group_by": ["PkgName"]
        }
      ]
    }
  ]
}
```
Notes:
- JSON outputs are arrays; combined JSON destinations produce a single concatenated array (see [Combining & Grouping](../output/combining-grouping.md)).
- `destination` controls where output goes; `git:pr` ignored unless `CI_EVENT=pr`.
- Empty sections omitted automatically.

## AI Governance Angle
Early PR integration detects:
- Newly introduced vulnerable dependencies
- License drift (e.g., Copyleft additions)
- Missing required files / invalid content (validation policies)
- High‑risk changes hidden in large automated diffs

## Troubleshooting
Centralized: see [Troubleshooting & FAQ](../operations/troubleshooting-faq.md). Unique GitHub issues:
| Issue | Cause | Fix |
|-------|-------|-----|
| 403 pulling image | Missing `packages: read` permission | Add permission |
| No PR comment | Missing `pull-requests: write` or event mismatch | Grant permission / verify `pull_request` event |
| No issue created on main scan | Event not `push` to default or permission missing | Ensure `issues: write` + default branch |

## Security Considerations
- Use the default `GITHUB_TOKEN` with least privileges; avoid storing personal tokens.
- Avoid `pull_request_target` with untrusted forks unless thoroughly reviewed.
- Keep dependency installs deterministic (`npm ci`, `pip install -r`, etc.) for reproducible results.

## Recommendations
- Pin action version (`@v0.0.1`) rather than `@latest`.
- Begin with `warn` actions for most categories; escalate to `block` after signal review.
- Use combined JSON + markdown for human + automation channels.

## Related Topics
- [Docker Setup](./docker.md)
- [Configuration Overview](../configuration/overview.md)
- [Combining & Grouping](../output/combining-grouping.md)
- [Starter Configs](../examples/starter-configs.md)

---
Next: refine policies — see the [Main Config Reference](../configuration/main-config.md).
