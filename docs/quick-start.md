# Quick Start

Get Codeward scanning your PRs in under 3 minutes.

## 1. Add the GitHub Action

Create `.github/workflows/codeward.yml`:

```yaml
name: Codeward Scan
on:
  pull_request:
  push:
    branches: [main]

jobs:
  scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: read
      pull-requests: write
      issues: write
    steps:
      - uses: codeward-io/scan@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

That's it. Commit and open a PR.

## 2. What Happens

**On pull requests**: Codeward compares your branch to `main` and posts a comment showing:
- New vulnerabilities introduced by your changes
- New license issues
- Package changes

**On push to main**: Codeward scans the current state and can create issues for existing problems.

## 3. Default Behavior

With no config file, Codeward uses sensible defaults:

- **Blocks** new CRITICAL, HIGH, and MEDIUM vulnerabilities
- **Blocks** new UNKNOWN, CRITICAL, and HIGH license issues  
- **Warns** about existing issues (doesn't block)
- **Posts** results to PR comments

## 4. Optional: Customize

Create `.codeward.json` in your repo root to customize behavior.

### Start Non-Blocking (Observe First)

```json
{
  "vulnerability": [{
    "name": "observe-all",
    "actions": { "new": "warn", "existing": "info" },
    "rules": [
      { "field": "Severity", "type": "eq", "value": "CRITICAL" },
      { "field": "Severity", "type": "eq", "value": "HIGH" }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
      "changes": ["new"]
    }]
  }]
}
```

### Block Only Critical

```json
{
  "vulnerability": [{
    "name": "block-critical",
    "actions": { "new": "block" },
    "rules": [
      { "field": "Severity", "type": "eq", "value": "CRITICAL" }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
      "changes": ["new"]
    }]
  }]
}
```

## 5. Next Steps

- **Add more policies**: Block risky licenses, track package changes → [Policies](./policies.md)
- **Configure outputs**: JSON files, webhooks, issue creation → [Outputs](./outputs.md)  
- **Full config reference**: All options explained → [Configuration](./configuration.md)
- **Use Docker**: For non-GitHub CI systems → [Docker Installation](./installation/docker.md)
