# Codeward Scan — GitHub Action

Scan your repository for policy, license, vulnerability, and validation issues before merge. Codeward provides diff-aware scanning (focus only on what changed) to govern AI-assisted and human code changes with transparent, deterministic outputs (Markdown / HTML / JSON).

- **Image**: `ghcr.io/codeward-io/scan:latest`
- **Action**: `codeward-io/scan@v0.2.0` (pin to a release)
- **Documentation**: [https://docs.codeward.io/](https://docs.codeward.io/)

## Quick start

Create a workflow at `.github/workflows/codeward-io-scan.yml` (minimal configuration — all inputs have defaults):

```yaml
name: Codeward
on:
  pull_request:
  workflow_dispatch:

jobs:
  scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read          # checkout
      packages: read          # pull GHCR image
      pull-requests: write    # comment on PRs
      issues: write           # (optional) create/update issues
    steps:
      - uses: codeward-io/scan@v0.2.0
```

The action will:
- Checkout base (main) and head (PR) automatically
- Pull the scanner image from GHCR
- Run a diff-aware scan (PR: base vs head, non-PR: current default branch)
- Produce findings and post a PR comment (if applicable)

For a fuller onboarding, see the [Quick Start Guide](https://docs.codeward.io/docs/quick-start).

## Inputs

All inputs have sane defaults; override only when needed.

- `event` (string, default: `${{ github.event_name }}`)
- `repository` (string, default: `${{ github.repository }}`)
- `current_branch` (string, default: `${{ github.ref }}`)
- `pr_number` (string, default: `${{ github.event.number }}`)
- `token` (string, default: `${{ github.token }}`)

Example with explicit inputs:

```yaml
- uses: codeward-io/scan@v0.2.0
  with:
    event: pull_request
    repository: my-org/my-repo
    token: ${{ secrets.GITHUB_TOKEN }}
```

## Configuration

Codeward is configured via a `.codeward.json` file in your repository root. If no config file exists, sensible defaults are used.

### Create a Configuration File

Create `.codeward.json` in your repository:

```json
{
  "vulnerability": [{
    "name": "block-critical",
    "actions": { "new": "block", "existing": "warn" },
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

This minimal config will:
- Block PRs that introduce new CRITICAL vulnerabilities
- Warn about existing CRITICAL vulnerabilities (doesn't block)
- Post results as a PR comment

### Key Features

- **Environment Variables**: Centralized `CODEWARD_*` variable management with config file overrides
- **Logging**: Structured, component-based logging with multiple formats and verbosity levels
- **Ignore Rules**: Global and policy-level ignores with optional expiration dates
- **Multiple Policies**: Separate policies for vulnerabilities, licenses, packages, validations, and PRs
- **Flexible Actions**: `info`, `warn`, or `block` for each change category (`new`, `existing`, `changed`, `removed`)
- **Advanced Validation**: Support for array wildcards (`*`) and custom output reasons
- **Conditional Logic**: `implies` operator for "If X then Y" policy logic

## Documentation

Complete documentation is available at **[https://docs.codeward.io/](https://docs.codeward.io/)**:

- **[Quick Start](https://docs.codeward.io/docs/quick-start)** - Get started in minutes
- **[Configuration](https://docs.codeward.io/docs/configuration)** - Complete config reference including:
  - Environment variables
  - Logging configuration
  - Ignore rules (global and policy-level)
  - Policy structure and rules
- **[Examples](https://docs.codeward.io/docs/examples)** - Real-world configuration examples
- **[Policies](https://docs.codeward.io/docs/policies)** - All policy types explained
- **[Outputs](https://docs.codeward.io/docs/outputs)** - Output formats and destinations
- **[Installation](https://docs.codeward.io/docs/installation/github-actions)** - GitHub Actions and Docker setup
- **[Troubleshooting](https://docs.codeward.io/docs/troubleshooting)** - Common issues and solutions

## Examples

### Block New Critical Vulnerabilities

```json
{
  "vulnerability": [{
    "name": "block-critical",
    "actions": { "new": "block", "existing": "warn" },
    "rules": [
      { "field": "Severity", "type": "eq", "value": "CRITICAL" }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "changes": ["new"]
    }]
  }]
}
```

### Block New Copyleft Licenses

```json
{
  "license": [{
    "name": "block-copyleft",
    "actions": { "new": "block" },
    "rules": [
      { "field": "Category", "type": "eq", "value": "Copyleft" }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "changes": ["new"]
    }]
  }]
}
```

### Ignore Test Directories

```json
{
  "global": {
    "ignore": [
      {
        "name": "Ignore test directories",
        "description": "Skip findings from test code",
        "paths": ["test/**", "**/test/**", "**/*_test.go"],
        "expires": "2026-12-31"
      }
    ]
  }
}
```

More examples: [https://docs.codeward.io/docs/examples](https://docs.codeward.io/docs/examples)

## Environment Variables

Configure Codeward using environment variables:

```yaml
- uses: codeward-io/scan@v0.2.0
  env:
    CODEWARD_LOG_LEVEL: debug
    CODEWARD_LOG_SUMMARY: detailed
```

**Key Variables:**
- `CODEWARD_LOG_LEVEL` - Log level (`silent`, `error`, `warn`, `info`, `debug`, `trace`)
- `CODEWARD_LOG_FORMAT` - Output format (`text`, `json`)
- `CODEWARD_LOG_SUMMARY` - Summary verbosity (`minimal`, `standard`, `detailed`)
- `CODEWARD_GITHUB_TOKEN` - GitHub token for API access

See the [Environment Variables documentation](https://docs.codeward.io/docs/configuration#environment-variables) for complete list.

## Support

- **Documentation**: [https://docs.codeward.io/](https://docs.codeward.io/)
- **Issues**: [GitHub Issues](https://github.com/codeward-io/scan/issues)
- **Website**: [https://codeward.io/](https://codeward.io/)

## License

See [LICENSE](LICENSE) file for details.


```yaml
- uses: codeward-io/scan@v0.2.0
  with:
    event: ${{ github.event_name }}
    repository: ${{ github.repository }}
    current_branch: ${{ github.ref }}
    pr_number: ${{ github.event.number }}
    token: ${{ secrets.GITHUB_TOKEN }}
```

## How it works
- PR events: checks out base + head, runs diff-mode (reduced noise; changed items only)
- Other events (push / workflow_dispatch): scans the checked-out default branch
- Caching: a writable cache mount accelerates repeated scans

Container mounts (conceptual):
- `./main` (base), `./branch` (PR head), `./results` (outputs), `/tmp/.cache` (scanner cache)

## Install dependencies for richer scanning (optional but recommended for license/vuln depth)
If you need dependency resolution (e.g. license transitive discovery), perform manual checkouts + installs, then run the container yourself for full control.

PR example:
```yaml
# (abbreviated) — full example in docs/installation/github-actions.md
- uses: actions/checkout@v4
  with:
    ref: ${{ github.base_ref }}
    path: main
- uses: actions/checkout@v4
  with:
    ref: ${{ github.head_ref }}
    path: branch
# install deps in both trees, then run the container as shown in docs
```

## Requirements
- Runner: `ubuntu-latest` (or self-hosted with Docker)
- Permissions (minimum):
  - `contents: read`
  - `packages: read`
  - `pull-requests: write` (for PR comments)
  - `issues: write` (optional)

## Troubleshooting
- GHCR auth failures: ensure `packages: read` granted; token scopes allow GHCR pull.
- Missing PR comment: confirm `pull-requests: write` and event is `pull_request` (not a fork with restricted token).
- Docker unavailable: use a runner with Docker pre-installed.
- Permissions on artifacts: container runs with host UID; ensure workspace writable.

## Learn more
Comprehensive documentation is available in the `docs/` directory:

### Getting Started
- **[Introduction](https://docs.codeward.io/docs/intro)** — Overview of Codeward, why use it, what it scans, and key features
- **[Quick Start](https://docs.codeward.io/docs/quick-start)** — Get up and running in 3 minutes with minimal configuration
- **[GitHub Actions Installation](https://docs.codeward.io/docs/installation/github-actions)** — Detailed GitHub Actions setup and integration
- **[Docker Installation](https://docs.codeward.io/docs/installation/docker)** — Run Codeward with Docker for local or CI/CD use

### Core Documentation
- **[Configuration](https://docs.codeward.io/docs/configuration)** — Complete reference for `.codeward.json` config file, global settings, and policy structure
- **[Policies](https://docs.codeward.io/docs/policies)** — All policy types explained: vulnerability, license, package, validation, and PR validation with examples and field references
- **[Outputs](https://docs.codeward.io/docs/outputs)** — Configure formats (markdown, HTML, JSON), destinations (PR comments, issues, files), templates, grouping, and combining outputs
- **[Examples](https://docs.codeward.io/docs/examples)** — Real-world configuration examples including ignore rules, environment variables, and logging
- **[Troubleshooting](https://docs.codeward.io/docs/troubleshooting)** — Common issues, environment variables, performance tips, and security notes

Narrative Kernel
"Codeward helps teams govern rapidly produced, AI-assisted code by enforcing security, license, and quality policies on every change. Diff-aware scanning highlights only what changed, reducing review noise and preventing silent risk introduction before merge."
