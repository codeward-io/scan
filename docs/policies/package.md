---
id: package-policies
title: Package Policies
description: Diff-aware, policy-driven dependency change gating: control new, changed, removed, existing packages with progressive enforcement & combined JSON inventory.
keywords:
  - dependencies
  - supply chain
  - package policies
  - diff-aware
  - progressive enforcement
---
<!-- filepath: /Users/tambet/Documents/GitHub/codeward-io/docs/docs/policies/package.md -->
# Package Policies

Package policies track dependency inventory changes (additions, removals, version updates, relationship shifts) and let you gate risky supply‑chain drift while observing existing backlog.

## Overview
Diff‑aware package policies categorize each dependency record as `new | changed | removed | existing` relative to the main branch (or baseline scan). See diff semantics: [Diff-Based Analysis](../concepts/diff-analysis.md). Actions let you escalate only net‑new risk (e.g. block unexpected new direct additions) while routing legacy churn to non‑blocking outputs. Rollout strategies: see [Progressive Enforcement](../operations/progressive-enforcement.md).

> Style & naming conventions (actions formatting `info | warn | block`, canonical change order) live in the [Style & Naming Guide](../configuration/style-naming-guide.md).

Canonical change category order everywhere: new, changed, removed, existing.

## Rationale / Principles
| Principle | Why it matters |
|-----------|----------------|
| Diff focus | Reduces noise by showing only what changed in a PR. |
| Progressive enforcement | Start with info/warn, graduate to block for unwanted patterns. |
| Narrow, composable policies | Easier to reason about than one mega policy. |
| Deterministic outputs | Stable automation inputs for inventory dashboards. |
| AI governance | AI‑assisted bulk updates often add dependencies— block only unacceptable new risk, not pre‑existing backlog. |

## Schema (Subset)
Full schema, operators, allowed fields: [Policy System](../concepts/policy-system.md). Naming guidance: [Style & Naming Guide](../configuration/style-naming-guide.md).
```json
{
  "package": [
    {
      "name": "direct-additions-gate",
      "actions": {"new": "warn", "removed": "info"},
      "rules": [ {"field": "Relationship", "type": "eq", "value": "direct"} ],
      "outputs": [
        {"format": "markdown", "template": "table", "destination": "git:pr", "fields": ["Name","Version","Relationship"], "changes": ["new"]}
      ]
    }
  ]
}
```
Notes:
* `rules` is an array (logical OR). No legacy object form.
* Display fields reference: [Allowed Record Fields](../concepts/policy-system.md#allowed-record-fields-filter--display).
* No `Ecosystem` field (legacy / not implemented) — remove if present.

## Common Rule Patterns
### Direct Dependencies Only
```json
{"rules": [ {"field":"Relationship","type":"eq","value":"direct"} ]}
```
### Exclude Internal Namespace (inequality)
```json
{"rules": [ {"field":"Name","type":"hasPrefix","value":"@"}, {"field":"Name","type":"ne","value":"@my-internal/core"} ]}
```
### Focus on Packages With Children (graph context)
```json
{"rules": [ {"field":"Children","type":"contains","value":""} ]}
```

## Example Policies
### New Package Review (PR emphasis)
```json
{"name":"new-package-review","actions":{"new":"warn"},
 "outputs":[{"format":"markdown","template":"table","destination":"git:pr","fields":["Name","Version","Relationship"],"changes":["new"],"collapse":true}]}
```
### Version Change Monitoring
```json
{"name":"version-changes","actions":{"changed":"info"},
 "outputs":[{"format":"markdown","template":"table","destination":"git:pr","fields":["Name","Version"],"changes":["changed"]}]}
```
### Block Direct Removals
```json
{"name":"protect-direct-removals","actions":{"removed":"block"},
 "rules":[{"field":"Relationship","type":"eq","value":"direct"}],
 "outputs":[{"format":"markdown","template":"table","destination":"git:pr","fields":["Name","Version","Relationship"],"changes":["removed"]}]}
```
### Combined JSON Inventory (automation)
```json
{"name":"inventory-json","actions":{"new":"info","changed":"info","removed":"info","existing":"info"},
 "outputs":[{"format":"json","destination":"file:/results/package-inventory.json","combined":true}]}
```

## Output Guidance
| Goal | Pattern |
|------|---------|
| PR signal only | Markdown table, changes:["new","removed","changed"] |
| Automation feed | Single combined JSON export (see [Combining & Grouping](../output/combining-grouping.md)) |
| Separate backlog | Send existing → file:/ or git:issue; exclude from PR comment |
| Reduced clutter | Omit fields not used in review decisions |

Combined JSON semantics: a single concatenated array containing all selected records across all matching policies (not one file per policy).

## Best Practices
| Objective | Recommendation |
|-----------|---------------|
| Gate risky drift | Start by blocking only new direct additions; monitor indirect via info outputs. |
| Minimize noise | Exclude `existing` from PR markdown; route to JSON/file instead. |
| Clarity | Keep each policy narrowly scoped (one intent). |
| Governance narrative | Add short `comment` explaining rationale for blockers. |
| Automation stability | Use combined JSON for ingestion; keep field set consistent. |

## Common Mistakes & Fixes
| Problem | Cause | Fix |
|---------|-------|-----|
| Rules ignored | Legacy `{ "rules": {"name": [...] }}` form | Convert to array of `{field,type,value}` |
| Unknown field `Ecosystem` | Field not implemented | Remove; use supported fields only |
| Everything shows as new | Missing baseline mount or wrong CI_EVENT | Provide `/main` + set `CI_EVENT=pr` |
| Mixed JSON/markdown in combined | Invalid format mixing | Use one format per combined destination |
| Relationship filter no effect | Dependency tree not built | Enable `global.dependency_tree=true` if filtering graph fields |

## Related / Next Steps
* Harden composition risk with [License Policies](./license.md)
* Add file & config assertions via [Validation Policies](./validation.md)
* Learn schema details: [Policy System](../concepts/policy-system.md)
* Plan staged rollout: [Progressive Enforcement](../operations/progressive-enforcement.md)
* Control outputs: [Output Formats](../output/formats.md) & [Combining & Grouping](../output/combining-grouping.md)

---
AI Governance: Package policies prevent unnoticed AI‑generated dependency additions from introducing unreviewed risk; diff gating ensures only net‑new packages can block merges.
