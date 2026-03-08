# Codeward Scan — GitHub Action

Scan your repository for policy, license, vulnerability, and validation issues before merge. Codeward provides diff-aware scanning (focus only on what changed) to govern AI-assisted and human code changes with transparent, deterministic outputs (Markdown / HTML / JSON).

- **Image**: `ghcr.io/codeward-io/scan:v0.3.0`
- **Action**: `codeward-io/scan@v0.3.0` (pin to a release)
- **Binaries**: [Download standalone binaries](https://github.com/codeward-io/scan/releases) for Linux, macOS, and Windows
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

#### Security Scanning
- **Vulnerability detection** — CVE-based scanning with severity levels (CRITICAL/HIGH/MEDIUM/LOW) and CVSS scoring
- **License detection** — Multi-source resolution (SPDX, package metadata, registry lookups, GitHub API) with persistent disk cache
- **Package analysis** — Dependency enumeration with direct/indirect tracking and parent/child relationships
- **17+ lockfile parsers** — Node.js, Python, Ruby, Go, Rust, PHP, .NET, Dart, Swift, Java, Elixir, C/C++ — no external dependencies
- **Direct Trivy DB queries** — OCI-based vulnerability database with automatic download, caching, and air-gapped support

#### Policy Engine
- **5 policy types** — Vulnerability, license, package, file validation, and PR validation
- **Flexible actions** — `info`, `warn`, or `block` per change category (`new`, `existing`, `changed`, `removed`)
- **Rich rule types** — `eq`, `ne`, `lt`, `gt`, `contains`, `regex`, `in`, `exists`, `last_match`, and more
- **Conditional logic** — `implies` operator for "If X then Y" policies
- **Global & policy-level ignores** — With optional expiration dates and author tracking

#### Diff Detection
- **Diff-aware scanning** — Compare base vs head to surface only what changed
- **Change categories** — New, removed, existing, and changed items with per-category actions
- **Multi-source diff** — Track changes per source path with glob pattern filtering

#### File & PR Validation
- **File validation** — JSON, YAML, TOML, env, properties files with auto-detection
- **Line-level scanning** — Scan individual lines for secrets/patterns with `LineNumber` and `MatchedContent` output
- **Cross-file references** — Validate values stay consistent across files (`ref_path`, `ref_key`, `ref_type`)
- **Array wildcards** — `spec.containers.*.image` with configurable match semantics (`all`, `any`, `none`)
- **Last match rules** — `last_match`/`not_last_match` with `line_filter` for multi-stage Dockerfile analysis
- **Per-line implies** — Combine `implies` + `scan: lines` for per-line "if X then Y" enforcement
- **PR metadata validation** — Title, body, labels, reviewers, file counts, changed files, and more
- **Conditional validation** — Check file content only when the file exists (implies + exists trigger)

#### Output & Reporting
- **Multiple destinations** — PR comments, GitHub issues, files, stdout/stderr, webhooks
- **Multiple formats** — Markdown, HTML, JSON with table/text/combined templates
- **Custom webhooks** — Send results to any HTTP endpoint (Slack, JIRA, PagerDuty, etc.) with template variables and secrets passthrough
- **Custom templates** — Load Go templates from filesystem for full output control
- **GroupBy & collapse** — Group results by field, collapse in PR comments

#### Configuration
- **YAML & JSON config** — Auto-discovered `.codeward.yaml` / `.codeward.yml` / `.codeward.json`
- **Strict validation** — Unknown fields rejected, enum validation, detailed error paths
- **Graceful degradation** — Invalid policies skipped with warnings, valid policies continue
- **CLI, env vars, and config files** — Hierarchical configuration with clear priority order

#### Ready-Made Profiles
Browse the **[Codeward Registry](https://github.com/codeward-io/registry)** for ready-to-use policy profiles covering security, infrastructure, language-specific checks, compliance, secrets detection, and more. Drop a profile into your repo and customize as needed.

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

## Standalone Binaries

Pre-built static binaries are available for systems where Docker isn't available or desired:

| OS | Architecture | Binary |
|----|-------------|--------|
| Linux | amd64, arm64 | `codeward-scan-linux-amd64` |
| macOS | amd64 (Intel), arm64 (Apple Silicon) | `codeward-scan-darwin-arm64` |
| Windows | amd64, arm64 | `codeward-scan-windows-amd64.exe` |

```bash
# Download, make executable, and run
curl -L -o codeward-scan \
  https://github.com/codeward-io/scan/releases/download/v0.3.0/codeward-scan-linux-amd64
chmod +x codeward-scan
./codeward-scan --config .codeward.yaml
```

Binaries are self-contained (static, `CGO_ENABLED=0`) with no runtime dependencies. Works with any CI system — see the [Standalone Binary docs](https://docs.codeward.io/docs/installation/standalone/) for GitLab CI and Azure Pipelines examples.

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
- [Standalone Binary](https://docs.codeward.io/docs/installation/standalone/) — No Docker required, runs on Linux/macOS/Windows
- [Kubernetes](https://docs.codeward.io/docs/installation/kubernetes/) — Job and CronJob setup
- [FAQ](https://docs.codeward.io/docs/faq/) — Common questions
- [Troubleshooting](https://docs.codeward.io/docs/troubleshooting/) — Common issues and solutions
- [Registry](https://github.com/codeward-io/registry) — Ready-to-use policy profiles for common use cases

## Support

- **Documentation**: [docs.codeward.io](https://docs.codeward.io/)
- **Registry**: [github.com/codeward-io/registry](https://github.com/codeward-io/registry)
- **Issues**: [GitHub Issues](https://github.com/codeward-io/scan/issues)
- **Website**: [codeward.io](https://codeward.io/)

## License

See [LICENSE](LICENSE) file for details.
