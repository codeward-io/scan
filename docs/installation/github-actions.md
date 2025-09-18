# GitHub Actions Integration

Integrate Codeward into your GitHub Actions workflows for automated security scanning on pull requests and pushes.

## Overview

Codeward doesn't have a pre-built GitHub Action, but it can be easily integrated into workflows using Docker or direct Go execution. This approach gives you full control over the scanning process and configuration.

## Basic Setup

### Simple Workflow

Create `.github/workflow/codeward-io-scan.yaml`:

```yaml
name: Codeward
on:
  workflow_dispatch:
  pull_request:
jobs:
  codeward-scan:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    permissions:
      pull-requests: write
      contents: read
      packages: read
      issues: write
    steps:
      - name: Scan
        uses: codeward-io/scan@latest
        with:
          event: ${{ github.event_name }}
          repository: ${{ github.repository }}
          current_branch: ${{ github.ref }}
          pr_number: ${{ github.event.number }}
```

### **Conditional Execution**

```yaml
name: Conditional Scan
on:
  pull_request:

jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      has-code-changes: ${{ steps.changes.outputs.code }}
      has-deps-changes: ${{ steps.changes.outputs.deps }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v2
        id: changes
        with:
          filters: |
            code:
              - 'src/**'
              - 'lib/**'
            deps:
              - 'package.json'
              - 'package-lock.json'
              - 'go.mod'
              - 'go.sum'
              - 'requirements.txt'

  codeward-scan:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    permissions:
      pull-requests: write
      contents: read
      packages: read
      issues: write
    needs: detect-changes
    if: needs.detect-changes.outputs.has-code-changes == 'true' || needs.detect-changes.outputs.has-deps-changes == 'true'
    runs-on: ubuntu-latest
    steps:
      - name: Scan
        uses: codeward-io/scan@v0.0.1
        with:
          event: ${{ github.event_name }}
          repository: ${{ github.repository }}
          current_branch: ${{ github.ref }}
          pr_number: ${{ github.event.number }}
```

### Configuration for GitHub Outputs

Update your `config.json` to include GitHub destinations:

```json
{
  "global": {
    "dependency_tree": false
  },
  "vulnerability": [{
    "name": "Critical security vulnerabilities",
    "disabled": false,
    "actions": {
      "new": "block",
      "existing": "warn",
      "removed": "info"
    },
    "rules": {
      "severity": [
        {"type": "eq", "value": "CRITICAL"},
        {"type": "eq", "value": "HIGH"}
      ]
    },
    "outputs": [
      {
        "format": "markdown",
        "template": "table",
        "destination": "git:pr",
        "title": "🔒 Security Vulnerabilities Found",
        "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
        "changes": ["new", "existing"],
        "collapse": true
      },
      {
        "format": "markdown", 
        "template": "text",
        "destination": "git:issue",
        "title": "🚨 Critical Security Issues Detected",
        "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
        "changes": ["existing"],
        "group_by": ["PkgName"]
      }
    ]
  }]
}
```

## 🔧 Troubleshooting

### **Common Issues**

**Permission Denied:**
```yaml
# Fix: Ensure proper permissions
permissions:
  contents: read
  pull-requests: write
  packages: read
  issues: write
```

**Timeout Issues:**
```yaml
# Fix: Increase timeout for large repositories
jobs:
  security-scan:
    timeout-minutes: 20  # Increased from default 10
```

## 📊 Monitoring and Metrics

### **Action Analytics**

Monitor workflow usage and performance:

```yaml
- name: Report Metrics
  if: always()
  run: |
    echo "Scan duration: ${{ steps.scan.outputs.duration }}"
    echo "Issues found: ${{ steps.scan.outputs.total_issues }}"
    echo "Blocked issues: ${{ steps.scan.outputs.blocked_issues }}"
```

## 🔐 Security Considerations

### **Token Management**

```yaml
# Use least-privilege tokens
- name: Security Scan
  uses: codeward-io/scan@v0.0.1
  with:
    token: ${{ secrets.GITHUB_TOKEN }}  # Built-in token with minimal permissions
```

---

**Next Steps:**
- Set up [Docker deployment](./docker.md) for custom environments
- Learn about [Configuration Management](../configuration/overview.md)

## Related Topics
- [Docker Setup](./docker.md)
- [Configuration Overview](../configuration/overview.md)
- [Output Destinations](../output/destinations.md)
