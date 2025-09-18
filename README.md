# Codeward Scan — GitHub Action

Scan your repository for policy, license, and security issues using Codeward.
This action runs the Codeward scanner Docker image against your code for PRs and main branch builds, and posts results back to your repository.

- Image: `ghcr.io/codeward-io/scan:latest`
- Action: `codeward-io/scan@v0.0.1` (pin to a release)

## Quick start
Create a workflow at `.github/workflows/codeward-io-scan.yml`:

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
      contents: read          # to checkout the repo (used by this action internally)
      packages: read          # to pull the GHCR image
      pull-requests: write    # to comment on PRs
      issues: write           # to create/update issues when needed
    steps:
      - uses: codeward-io/scan@v0.0.1
        # All inputs have sensible defaults from the GitHub context; no configuration required.
```

That’s it. The action will:
- Checkout your repository (base and head for PRs)
- Pull the scanner image from GHCR
- Run a scan (PR scans compare base vs. head; non-PR scans run on the default branch)

## Inputs
All inputs have defaults and can be omitted in most setups. Override only if you need custom behavior.

- `event` (string, default: `${{ github.event_name }}`): Current workflow event.
- `repository` (string, default: `${{ github.repository }}`): Owner/repo of the target repository.
- `current_branch` (string, default: `${{ github.ref }}`): Ref of the branch being scanned.
- `pr_number` (string, default: `${{ github.event.number }}`): Pull request number for PR scans.
- `token` (string, default: `${{ github.token }}`): GitHub token used to pull the GHCR image and post results.

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
- On pull requests, the action checks out the base and head, mounts both into the container, and runs a diff-aware scan.
- On non-PR events (push, workflow_dispatch), it scans the default branch checkout.
- A cache directory is mounted to improve subsequent runs.

Under the hood, the action executes a Docker container with mounts similar to:
- `./main` (base), `./branch` (head, for PRs), `./results` (outputs), `./cache` (scanner cache)

## Requirements
- Runner: `ubuntu-latest` (or a self-hosted runner with Docker available)
- Permissions (at minimum):
  - `contents: read`
  - `packages: read` (required to pull from `ghcr.io`)
  - `pull-requests: write` (to post PR comments)
  - `issues: write` (optional, for issue updates)

## Troubleshooting
- GHCR pull fails (403/401): Ensure the job has `packages: read` and that the GITHUB_TOKEN has permission to read GHCR in your org.
- No PR comments appear: Ensure `pull-requests: write` is granted. For forks, consider your security posture; `pull_request_target` runs with elevated permissions, use with care.
- Docker not available: Use `ubuntu-latest` or install Docker on your self-hosted runner.
- File permission issues on artifacts: The container runs as the host UID; ensure the workspace is writable by that user.

## Learn more
- Introduction: [docs/intro.md](docs/intro.md)
- Install via GitHub Actions: [docs/installation/github-actions.md](docs/installation/github-actions.md)
- Run with Docker locally: [docs/installation/docker.md](docs/installation/docker.md)
- Starter configurations: [docs/examples/starter-configs.md](docs/examples/starter-configs.md)
- Output: [formats](docs/output/formats.md), [destinations](docs/output/destinations.md)
- Policies: [docs/policies/](docs/policies/)
- Concepts: [docs/concepts/](docs/concepts/)
- Configuration: [overview](docs/configuration/overview.md), [main config](docs/configuration/main-config.md)
- AI governance: [overview](docs/ai-governance/overview.md), [best practices](docs/ai-governance/best-practices.md), [policy enforcement](docs/ai-governance/policy-enforcement.md), [security risks](docs/ai-governance/security-risks.md)
