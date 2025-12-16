# Welcome to Codeward

**Codeward** is a policy-driven security scanner that governs code changes before they merge. It detects vulnerabilities, license issues, package changes, and policy violations — focusing only on what's new or changed so reviewers aren't overwhelmed by existing backlog.

## Why Codeward?

Modern development is fast. AI-assisted code generation makes it faster. This velocity increases the risk of silently introducing:

- **Vulnerable dependencies** with known CVEs
- **Incompatible licenses** that create legal exposure  
- **Unwanted packages** that expand attack surface
- **Policy violations** in configs, Dockerfiles, or CI workflows

Codeward catches these issues automatically, explains what's wrong, and can block merges when critical risks are introduced.

## What It Scans

| Scan Type | What It Detects |
|-----------|-----------------|
| **Vulnerabilities** | CVEs in dependencies (via embedded Trivy) with severity, fix versions |
| **Licenses** | License names, categories, and risk levels for compliance |
| **Packages** | New, removed, or changed dependencies between branches |
| **Validations** | Custom rules on files (text, JSON, YAML) and filesystem structure |
| **PR Validations** | Rules on PR metadata, size, files changed, and patch content |

## How It Works

1. **Scan** both branches (main and feature) for vulnerabilities, licenses, and packages
2. **Diff** the results to classify each finding: `new`, `changed`, `removed`, or `existing`
3. **Filter** findings through your policy rules
4. **Act** on each category with `info`, `warn`, or `block`
5. **Report** results to PR comments, issues, files, or webhooks

The key insight: **only `new` findings can block your PR**, so existing technical debt doesn't slow you down while you address it separately.

## Key Features

- **Diff-aware**: Focus on net-new risk, not legacy backlog
- **Policy-as-code**: Define rules in a single JSON config file  
- **Flexible actions**: `info` (log only), `warn` (visible but passes), `block` (fails CI)
- **Multiple outputs**: PR comments, GitHub issues, JSON files, webhooks
- **Zero config option**: Works out-of-box with sensible defaults
- **Deterministic**: Same inputs produce identical outputs for automation

## Quick Example

Block new critical vulnerabilities in PRs:

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

This policy:
- **Blocks** PRs that introduce new critical vulnerabilities
- **Warns** about existing critical vulnerabilities (doesn't block)
- **Posts** a markdown table to the PR comment showing only new issues

## Get Started

**Fastest path**: Add the GitHub Action with zero config → [Quick Start](./quick-start.md)

**Choose your installation**:
- [GitHub Actions](./installation/github-actions.md) — easiest for GitHub repos
- [Docker](./installation/docker.md) — works with any CI system

**Learn more**:
- [Configuration](./configuration.md) — full config reference
- [Policies](./policies.md) — all policy types explained
- [Outputs](./outputs.md) — formats, destinations, combining

---

**Stop reviewing security issues manually.** Let Codeward surface what matters and block what's critical.
