---
title: Security & Trust Model
description: Data handling, permissions, network egress, and threat considerations for running Codeward in CI.
---

# Security & Trust Model

Transparency for security reviewers and platform owners.

## Data Handling
- All scanning occurs within the CI job container.
- No source code leaves the runner by default.
- Outputs written only to configured destinations (comment, files, stdout, optional webhook `url`).

## Network Egress
- Default: no external network calls except package index queries performed by underlying tools (if any).
- Optional: `url` destination performs an HTTPS POST with JSON payload.

## Tokens & Permissions
- GitHub Action uses `GITHUB_TOKEN` with minimal scopes required to comment on PRs.
- No long-lived secrets stored; permissions inherited from workflow context.

## Cache Contents
- Dependency metadata and intermediate scan results (non-sensitive derived data) may be cached to speed re-scans.
- No credential material cached by Codeward.

## Privacy & Residency
- Data resides ephemerally on the CI runner; when the job ends, artifacts are discarded unless explicitly persisted.

## Threat Considerations
| Vector | Mitigation |
|--------|-----------|
| Exfiltration via destination | User controls destinations; restrict `url` to trusted endpoints |
| False positives blocking dev | Progressive enforcement strategy; start in `warn` mode |
| Tampering with outputs | Deterministic renderer; changes require config update |

## Out of Scope (Current)
- SARIF export (design trade-off: noise vs signal)
- Inline suppression annotations (roadmap consideration)

See also: [Progressive Enforcement](progressive-enforcement.md), [Troubleshooting & FAQ](troubleshooting-faq.md), [Architecture](../concepts/architecture.md).
