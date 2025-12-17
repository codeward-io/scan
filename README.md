# Codeward Scan — GitHub Action

Scan your repository for policy, license, vulnerability, and validation issues before merge. Codeward provides diff-aware scanning (focus only on what changed) to govern AI-assisted and human code changes with transparent, deterministic outputs (Markdown / HTML / JSON).

- Image: `ghcr.io/codeward-io/scan:latest`
- Action: `codeward-io/scan@v0.0.1` (pin to a release)

## Quick start
Create a workflow at `.github/workflows/codeward-io-scan.yml` (minimal configuration — all inputs have defaults):

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
      contents: read          # checkout
      packages: read          # pull GHCR image
      pull-requests: write    # comment on PRs
      issues: write           # (optional) create/update issues
    steps:
      - uses: codeward-io/scan@v0.0.1
```

The action will:
- Checkout base (main) and head (PR) automatically
- Pull the scanner image from GHCR
- Run a diff-aware scan (PR: base vs head, non-PR: current default branch)
- Produce findings and post a PR comment (if applicable)

For a fuller onboarding (including optional dependency install patterns) see: `docs/quick-start.md`.

## Inputs
All inputs have sane defaults; override only when needed.

- `event` (string, default: `${{ github.event_name }}`)
- `repository` (string, default: `${{ github.repository }}`)
- `current_branch` (string, default: `${{ github.ref }}`)
- `pr_number` (string, default: `${{ github.event.number }}`)
- `token` (string, default: `${{ github.token }}`)

Example with explicit inputs:

```yaml
- uses: codeward-io/scan@v0.0.1
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
- `./main` (base), `./branch` (PR head), `./results` (outputs), `./cache` (scanner cache)

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
- **[Introduction](docs/intro.md)** — Overview of Codeward, why use it, what it scans, and key features
- **[Quick Start](docs/quick-start.md)** — Get up and running in 3 minutes with minimal configuration
- **[GitHub Actions Installation](docs/installation/github-actions.md)** — Detailed GitHub Actions setup and integration
- **[Docker Installation](docs/installation/docker.md)** — Run Codeward with Docker for local or CI/CD use

### Core Documentation
- **[Configuration](docs/configuration.md)** — Complete reference for `.codeward.json` config file, global settings, and policy structure
- **[Policies](docs/policies.md)** — All policy types explained: vulnerability, license, package, validation, and PR validation with examples and field references
- **[Outputs](docs/outputs.md)** — Configure formats (markdown, HTML, JSON), destinations (PR comments, issues, files), templates, grouping, and combining outputs
- **[Troubleshooting](docs/troubleshooting.md)** — Common issues, environment variables, performance tips, and security notes

Narrative Kernel
"Codeward helps teams govern rapidly produced, AI-assisted code by enforcing security, license, and quality policies on every change. Diff-aware scanning highlights only what changed, reducing review noise and preventing silent risk introduction before merge."
