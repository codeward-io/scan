---
id: intro
title: Introduction
description: Codeward diff-aware policy engine: govern vulnerabilities, licenses, packages & validation changes with deterministic markdown & JSON outputs before merge.
keywords:
  - codeward
  - diff-aware
  - policy engine
  - vulnerabilities
  - licenses
  - validation
  - supply chain security
---
# Welcome to Codeward

**Codeward** helps you govern code changes — human or AI‑generated — before they merge. It performs policy‑driven analysis (vulnerabilities, licenses, package diffs, and custom validations) with diff awareness so you focus only on what changed and the risk it introduces.

## Executive TL;DR
- Diff-aware policies gate only net-new or modified risk (no legacy backlog noise in PRs).
- Single configuration governs vulnerabilities, licenses, packages, and validations with `info | warn | block` actions.
- Deterministic Markdown & JSON outputs (concatenated arrays) for both reviewers and automation.

## Why Codeward (Now)
Modern development velocity and AI‑assisted generation increase the risk of silently adding vulnerable dependencies, incompatible licenses, or violating internal quality rules. Codeward inserts an automated, explainable governance layer into your workflow so risky changes are surfaced early and (if you wish) blocked.

## What Codeward Scans
- **Vulnerabilities** (embedded Trivy) across packages
- **Licenses** with severity / category for compliance posture
- **Package Changes** (new / removed / changed / existing) between main and a feature branch
- **Custom Validations** (text / JSON / YAML / filesystem existence rules) for project conventions

## How It Works (High Level)
1. (PRs) Compare main vs feature branch → classify changes: new, changed, removed, existing (see [Diff-Based Analysis](./concepts/diff-analysis.md)).
2. Apply configured policies (filters + actions per change type — see [Policy System](./concepts/policy-system.md)).
3. Generate reports (Markdown, HTML, JSON array) — optionally grouped or combined.
4. Deliver to destinations: files, logs, PR comment, or issue (event‑aware).
5. Enforce gates: any section with a `block` action sets a non‑zero exit.

## Key Benefits
- **Diff‑Aware Noise Reduction**: Focus reviewers on the delta — not legacy backlog.
- **Policy Gates**: Consistent, codified rules (security, license, validation) with `info | warn | block` actions per change category.
- **AI Governance Support**: Surfaces license drift, new vulnerable libs, and broken conventions often introduced by automated code generation.
- **Deterministic Outputs**: Stable JSON array schema for automation; clean Markdown/HTML for humans.
- **Multi‑Destination Delivery**: PR comment, issue, logs, or files — configurable per policy.
- **Composable Configuration**: Fine‑grained field selection, grouping, change filtering, combined reports.

## Quick Start
Choose an integration path:
- **GitHub Actions**: Use the published action for automatic diff scans. See [GitHub Actions installation](./installation/github-actions.md).
- **Docker (any CI)**: Run the container directly. See [Docker installation](./installation/docker.md).

Then explore policies and configuration:
- Scanning concepts: [Scanning Types](./concepts/scanning-types.md)
- Policy model: [Policy System](./concepts/policy-system.md)
- Configuration overview: [Configuration Overview](./configuration/overview.md)
- Main configuration reference: [Main Config](./configuration/main-config.md)

## Policy & Actions Overview
Removed redundancy — canonical definitions live in [Diff-Based Analysis](./concepts/diff-analysis.md) and [Policy System](./concepts/policy-system.md).

## Reporting
- **Formats**: Markdown, HTML, JSON (uniform array schema)
- **Templates** (non‑JSON): table or text
- **Customization**: Select fields, group related findings, filter by change categories, combine multiple policies into one artifact.
See: [Output Formats](./output/formats.md) and [Destinations](./output/destinations.md).

## Production Readiness Features
- Deterministic ordering & severity handling
- Caching for faster re‑runs (Trivy + internal cache)
- Event‑aware GitHub posting (PR comment vs issue)
- Clear non‑features (no SARIF, no auto‑remediation) to avoid surprise

## Next Steps
1. Install via your preferred method (Actions or Docker).
2. Start with starter configs: [Starter Configs](./examples/starter-configs.md).
3. Tailor or add policies (license, vulnerability, package, validation).
4. Add or tighten `block` actions to enforce standards once confident.

---
**Move from reactive reviews to proactive governance.** Begin with the [GitHub Actions guide](./installation/github-actions.md) or the [Docker guide](./installation/docker.md).
