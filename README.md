# Codeward Scan — GitHub Action

Scan your repository for policy, license, vulnerability, and validation issues before merge. Codeward provides diff-aware scanning (focus only on what changed) to govern AI-assisted and human code changes with transparent, deterministic outputs (Markdown / HTML / JSON).

- **Image**: `ghcr.io/codeward-io/scan:v0.3.0`
- **Action**: `codeward-io/scan@v0.3.0` (pin to a release)
- **Documentation**: [https://docs.codeward.io/](https://docs.codeward.io/)

## Quick Start

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
      - uses: codeward-io/scan@v0.3.0
```

The action will:
- Checkout base (main) and head (PR) automatically
- Pull the scanner image from GHCR
- Run a diff-aware scan (PR: base vs head, non-PR: current default branch)
- Produce findings and post a PR comment (if applicable)

For a fuller onboarding, see the [Quick Start Guide](https://docs.codeward.io/docs/quick-start/).

## Inputs

All inputs have sane defaults; override only when needed.

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `event` | Yes | `${{ github.event_name }}` | Event name |
| `repository` | Yes | `${{ github.repository }}` | Repository |
| `current_branch` | Yes | `${{ github.ref }}` | Current branch |
| `pr_number` | No | `${{ github.event.number }}` | Pull request number |
| `token` | Yes | `${{ github.token }}` | GitHub token |
| `webhook_secrets` | No | — | Multiline `KEY=VALUE` pairs for custom webhook templates |

### Webhook Secrets

Pass sensitive values for [custom webhooks](https://docs.codeward.io/docs/outputs/#custom-webhooks) via the `webhook_secrets` input. Each line is a `KEY=VALUE` pair exported as an environment variable available in webhook templates.

```yaml
- uses: codeward-io/scan@v0.3.0
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    webhook_secrets: |
      SLACK_TOKEN=${{ secrets.SLACK_TOKEN }}
      JIRA_API_KEY=${{ secrets.JIRA_API_KEY }}
      PAGERDUTY_KEY=${{ secrets.PAGERDUTY_KEY }}
```

Secrets are parsed by the entrypoint, exported individually, and the raw value is unset — never leaked downstream. Reference them in webhook config using `{{.Env.SLACK_TOKEN}}`.

## How It Works

- **PR events**: Checks out base + head, runs diff-mode (only changed items)
- **Other events** (push / workflow_dispatch): Scans the default branch
- **Caching**: Writable cache mount (`/tmp/.cache`) accelerates repeated scans

## Configuration

Codeward is configured via a `.codeward.yaml` (or `.codeward.json`) file in your repository root. If no config file exists, sensible defaults are used.

```yaml
# .codeward.yaml
vulnerability:
  - name: block-critical
    actions:
      new: block
      existing: warn
    rules:
      - field: Severity
        type: eq
        value: CRITICAL
    outputs:
      - format: markdown
        destination: "git:pr"
        fields: [VulnerabilityID, PkgName, Severity, FixedVersion]
        changes: [new]
```

### Key Features

- **YAML & JSON config** — Auto-discovered `.codeward.yaml` / `.codeward.yml` / `.codeward.json`
- **CVSS scoring** — Filter vulnerabilities by `CVSSScore` (numeric ranges) and `CVSSVector`
- **Multi-source license resolution** — SPDX, package metadata, registry lookups with disk cache
- **File validation** — JSON, YAML, TOML, env, properties files with auto-detection
- **Line-level scanning** — Scan individual lines for secrets, patterns, and per-line policy enforcement
- **Cross-file references** — Validate values stay consistent across files (`ref_path`, `ref_key`)
- **Custom webhooks** — Send results to any HTTP endpoint (Slack, JIRA, PagerDuty, etc.)
- **Conditional logic** — `implies` operator for "If X then Y" policies
- **Flexible actions** — `info`, `warn`, or `block` per change category (`new`, `existing`, `changed`, `removed`)
- **Global & policy-level ignores** — With optional expiration dates

## Examples

### Block Critical Vulnerabilities

```yaml
vulnerability:
  - name: block-critical
    actions: { new: block, existing: warn }
    rules:
      - field: Severity
        type: eq
        value: CRITICAL
    outputs:
      - format: markdown
        destination: "git:pr"
        changes: [new]
```

### Block Copyleft Licenses

```yaml
license:
  - name: block-copyleft
    actions: { new: block }
    rules:
      - field: Category
        type: eq
        value: Copyleft
    outputs:
      - format: markdown
        destination: "git:pr"
        changes: [new]
```

### Ignore Test Directories

```yaml
global:
  ignore:
    - name: Ignore test directories
      description: Skip findings from test code
      paths: ["test/**", "**/test/**", "**/*_test.go"]
      expires: "2026-12-31"
```

More examples: [docs.codeward.io/docs/examples](https://docs.codeward.io/docs/examples/)

## Environment Variables

```yaml
- uses: codeward-io/scan@v0.3.0
  env:
    CODEWARD_LOG_LEVEL: debug
    CODEWARD_LOG_SUMMARY: detailed
```

| Variable | Description |
|----------|-------------|
| `CODEWARD_LOG_LEVEL` | Log level: `silent`, `error`, `warn`, `info`, `debug`, `trace` |
| `CODEWARD_LOG_FORMAT` | Output format: `text`, `json` |
| `CODEWARD_LOG_SUMMARY` | Summary verbosity: `minimal`, `standard`, `detailed` |
| `CODEWARD_CACHE_DIR` | Cache directory path |
| `TRIVY_SKIP_DB_UPDATE` | Skip Trivy DB download (air-gapped mode) |
| `CODEWARD_GITHUB_TOKEN` | GitHub token for API access |

See the [CLI & Environment Variables](https://docs.codeward.io/docs/cli-and-env/) reference for the complete list.

## Install Dependencies (Optional)

For richer license and vulnerability detection, install dependencies before scanning. Perform manual checkouts + installs, then run the container directly:

```yaml
# abbreviated — full example in docs
- uses: actions/checkout@v4
  with: { ref: "${{ github.base_ref }}", path: main }
- uses: actions/checkout@v4
  with: { ref: "${{ github.head_ref }}", path: branch }
- run: npm ci
  working-directory: main
- run: npm ci
  working-directory: branch
# then run the scanner container — see Installation docs
```

See [GitHub Actions Installation](https://docs.codeward.io/docs/installation/github-actions/) for the full example.

## Requirements

- **Runner**: `ubuntu-latest` (or self-hosted with Docker)
- **Permissions**:
  - `contents: read` — checkout
  - `packages: read` — pull GHCR image
  - `pull-requests: write` — PR comments (if using `git:pr`)
  - `issues: write` — issues (if using `git:issue`)

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| GHCR auth failures | Ensure `packages: read` permission; token scopes allow GHCR pull |
| Missing PR comment | Confirm `pull-requests: write` and event is `pull_request` |
| Docker unavailable | Use a runner with Docker pre-installed |
| Permission errors on artifacts | Container runs with host UID; ensure workspace writable |

See [Troubleshooting](https://docs.codeward.io/docs/troubleshooting/) for more solutions.

## Documentation

- [Introduction](https://docs.codeward.io/docs/intro/) — Overview and core philosophy
- [Quick Start](https://docs.codeward.io/docs/quick-start/) — Get running in 3 minutes
- [Configuration](https://docs.codeward.io/docs/configuration/) — Complete config reference (YAML/JSON)
- [CLI & Environment Variables](https://docs.codeward.io/docs/cli-and-env/) — All CLI options and env vars
- [Policies](https://docs.codeward.io/docs/policies/) — Vulnerability, license, package, validation, and PR policies
- [Outputs](https://docs.codeward.io/docs/outputs/) — Formats, destinations, templates, and custom webhooks
- [Examples](https://docs.codeward.io/docs/examples/) — Real-world configuration examples
- [GitHub Actions](https://docs.codeward.io/docs/installation/github-actions/) — Detailed GitHub Actions setup
- [Docker](https://docs.codeward.io/docs/installation/docker/) — Docker usage for any CI
- [Kubernetes](https://docs.codeward.io/docs/installation/kubernetes/) — Job and CronJob setup
- [FAQ](https://docs.codeward.io/docs/faq/) — Common questions
- [Troubleshooting](https://docs.codeward.io/docs/troubleshooting/) — Common issues and solutions

## Support

- **Documentation**: [docs.codeward.io](https://docs.codeward.io/)
- **Issues**: [GitHub Issues](https://github.com/codeward-io/scan/issues)
- **Website**: [codeward.io](https://codeward.io/)

## License

See [LICENSE](LICENSE) file for details.
