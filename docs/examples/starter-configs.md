# Starter Configurations

Opinionated starting points for adopting Codeward with progressive, diff‑aware enforcement. Copy, trim, or extend — keep each policy focused.

## Overview
These staged examples show how to: (1) gate only net‑new critical risk first, (2) layer observation for broader risk, (3) introduce license & package governance, (4) add validation. All assume diff (PR) scanning: highlight `new` (and where helpful `changed`) while routing backlog elsewhere.

Reviewer expectations (use these in internal docs / runbooks):
- Stage 1: Review only blocked CRITICAL vulnerabilities introduced by the PR.
- Stage 2: Continue blocking CRITICAL; review HIGH entries (warn) for prioritization.
- Stage 3: In addition, ensure no prohibited licenses (GPL) newly appear; glance at new direct dependencies.
- Stage 4: Begin noting structural validation findings (warn) to prepare for future blocking.
- Stage 5+: Promote selected validations or HIGH severity to block based on remediation progress.

## Principles
| Principle | Applied Here |
|-----------|--------------|
| Diff focus | PR markdown restricted to `new` (occasionally `changed`). Backlog routed to JSON/file. |
| Progressive rollout | Start blocking only CRITICAL (or Copyleft) → layer warnings → escalate. |
| Narrow policies | One intent per policy (critical block, high observe, license GPL block, new direct packages, etc.). |
| Deterministic outputs | Stable tables for PR; combined JSON for automation. |
| AI governance | Gates net‑new high risk from large or AI‑generated commits. |

---
## 1. Minimal Production Gate (Block New Critical Vulnerabilities)
```json
{
  "vulnerability": [
    {
      "name": "block-critical-new",
      "actions": {"new": "block", "existing": "warn"},
      "rules": [ {"field": "Severity", "type": "eq", "value": "CRITICAL"} ],
      "outputs": [
        {"format": "markdown", "template": "table", "destination": "git:pr", "fields": ["VulnerabilityID","PkgName","Severity","FixedVersion"], "changes": ["new"], "collapse": true}
      ]
    }
  ]
}
```
Purpose: Block net‑new critical issues. Existing backlog warns (non‑blocking) until a remediation plan exists.

---
## 2. Add High Severity Observation
```json
{
  "vulnerability": [
    {"name": "block-critical-new", "actions": {"new":"block","existing":"warn"}, "rules": [ {"field":"Severity","type":"eq","value":"CRITICAL"} ],
     "outputs": [ {"format":"markdown","template":"table","destination":"git:pr","fields":["VulnerabilityID","PkgName","Severity","FixedVersion"],"changes":["new" ]} ]},
    {"name": "observe-high-new", "actions": {"new":"warn"}, "rules": [ {"field":"Severity","type":"eq","value":"HIGH"} ],
     "outputs": [ {"format":"markdown","template":"table","destination":"git:pr","fields":["VulnerabilityID","PkgName","Severity"],"changes":["new"],"collapse": true} ]}
  ]
}
```
Purpose: High severity additions surface (warn) without blocking; CRITICAL still blocks.

---
## 3. License & Package Awareness Layer
Add license gating for GPL and visibility into new direct dependencies.
```json
{
  "vulnerability": [
    {"name":"block-critical-new","actions":{"new":"block","existing":"warn"},"rules":[{"field":"Severity","type":"eq","value":"CRITICAL"}],
     "outputs":[{"format":"markdown","template":"table","destination":"git:pr","fields":["VulnerabilityID","PkgName","Severity","FixedVersion"],"changes":["new"]}]}
  ],
  "license": [
    {"name":"gpl-block","actions":{"new":"block","existing":"warn"},"rules":[{"field":"Category","type":"eq","value":"Copyleft"}],
     "outputs":[{"format":"markdown","template":"table","destination":"git:pr","fields":["Name","Category","Severity","PkgName"],"changes":["new"],"collapse":true}]}
  ],
  "package": [
    {"name":"direct-new-observe","actions":{"new":"info"},"rules":[{"field":"Relationship","type":"eq","value":"direct"}],
     "outputs":[{"format":"markdown","template":"table","destination":"git:pr","fields":["Name","Version","Relationship"],"changes":["new"]}]}
  ]
}
```

---
## 4. Combined JSON Automation Output
Adds a single JSON artifact combining multiple policy outputs (all JSON at same destination). Combined JSON semantics = one concatenated array (no wrapper). See [Combining & Grouping](../output/combining-grouping.md).
```json
{
  "vulnerability": [
    {"name":"crit-block","actions":{"new":"block"},"rules":[{"field":"Severity","type":"eq","value":"CRITICAL"}],
     "outputs":[{"format":"json","destination":"file:/results/security.json","combined":true,"changes":["new"]}]},
    {"name":"high-observe","actions":{"new":"warn"},"rules":[{"field":"Severity","type":"eq","value":"HIGH"}],
     "outputs":[{"format":"json","destination":"file:/results/security.json","combined":true,"changes":["new"]}]}
  ],
  "license": [
    {"name":"gpl-block","actions":{"new":"block"},"rules":[{"field":"Category","type":"eq","value":"Copyleft"}],
     "outputs":[{"format":"json","destination":"file:/results/security.json","combined":true,"changes":["new"]}]}
  ]
}
```
Purpose: Single machine‑readable artifact for pipelines or dashboards.

---
## 5. Adding Validation Policies (Config Hardening)
```json
{
  "validation": [
    {"name":"dockerfile-user","type":"text","path":"Dockerfile","actions":{"new":"block","changed":"block","existing":"warn"},
     "rules":[{"type":"contains","value":"USER"}],
     "outputs":[{"format":"markdown","template":"table","destination":"git:pr","fields":["key","reason","passing"],"changes":["new","changed"],"collapse":true}]},
    {"name":"workflow-permissions","type":"yaml","path":".github/workflows/*.yml","actions":{"new":"block","changed":"block"},
     "rules":[{"type":"eq","key":"permissions.contents","value":"read"},{"type":"not_contains","key":"jobs.*.steps[*].run","value":"curl | sh"}],
     "outputs":[{"format":"markdown","template":"table","destination":"git:pr","fields":["key","reason","passing"],"changes":["new","changed"]}]}
  ]
}
```
Purpose: Enforce non‑root Docker builds & minimal workflow permissions.

---
## Progressive Rollout
Detailed phased guidance lives in the dedicated [Progressive Enforcement](../operations/progressive-enforcement.md) page.

## Best Practices
| Objective | Recommendation |
|-----------|---------------|
| Keep PR review focused | Limit PR markdown outputs to `new` (and essential `changed`). |
| Avoid config sprawl | Reuse destinations; favor multiple small policies. |
| Ensure determinism | Fix field ordering; prefer consistent table column order. |
| Document rationale | Use `comment` or `title` to explain gating decisions. |
| Separate backlog | Route `existing` to JSON (file destination) rather than PR. |

## Common Mistakes & Fixes
| Problem | Cause | Fix |
|---------|-------|-----|
| Nested rules object used | Legacy schema | Use array: `"rules": [{"field":...}]` |
| License field `license` invalid | Not an allowed field | Use `Name`, `Category`, or `Severity` |
| Mixed formats in combined JSON | Combined group had markdown output | Keep combined groups JSON only |
| Validation single `action` key | Misapplied prior pattern | Provide `actions` map keyed by change category |
| Backlog noise in PR | Included `existing` | Limit `changes` to `new` (and necessary `changed`) |

## Related / Next Steps
- Policy model: [Policy System](../concepts/policy-system.md)
- Diff categorization: [Diff-Based Analysis](../concepts/diff-analysis.md)
- Output rules: [Combining & Grouping](../output/combining-grouping.md)
- Validation details: [Validation Policies](../policies/validation.md)

---
AI Governance: These staged configs let you gate only net‑new critical & license risk while incrementally layering additional rules, preventing large or AI‑generated commits from silently expanding attack surface or compliance debt.
