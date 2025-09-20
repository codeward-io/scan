---
title: License Policies
description: Configure diff-aware license policies to block net-new prohibited or high-risk licenses while observing existing backlog.
keywords: license policies, software composition, license risk, diff-aware policies, compliance, AI governance, policy schema, rule patterns, output strategy, exceptions, best practices
---

# License Policies

Govern software composition license risk (prohibited / reciprocal / high‑risk categories) with diff‑aware policies that block net‑new problematic licenses while observing existing backlog.

## Overview
License policies filter detected package licenses and apply actions per change category (`new | changed | removed | existing`). Use multiple focused policies (e.g., block GPL, observe LGPL) to progressively tighten compliance without stalling delivery. See [Progressive Enforcement](../operations/progressive-enforcement.md) for staged rollout patterns. Canonical change semantics: [Diff-Based Analysis](../concepts/diff-analysis.md).

> Style & naming conventions (actions formatting `info | warn | block`, change category order) live in the [Style & Naming Guide](../configuration/style-naming-guide.md).

## AI Governance Rationale
Fast AI‑assisted dependency additions can quietly introduce reciprocal or prohibited licenses. Diff focus ensures only new or changed risky licenses block while legacy debt is surfaced separately for planned remediation.

## Policy Schema (Subset)
Full schema & operators: [Policy System](../concepts/policy-system.md). Allowed record fields: [Allowed Fields](../concepts/policy-system.md#allowed-record-fields-filter--display). Naming conventions: [Style & Naming Guide](../configuration/style-naming-guide.md).
```json
{
  "name": "block-gpl",
  "actions": {"new": "block", "existing": "warn"},
  "rules": [ {"field": "Name", "type": "contains", "value": "GPL"} ],
  "outputs": [
    {"format": "markdown", "template": "table", "destination": "git:pr", "fields": ["Name","Category","Severity","PkgName"], "changes": ["new"], "collapse": true},
    {"format": "json", "destination": "file:/results/license-new.json", "changes": ["new"], "combined": true}
  ]
}
```
Notes:
- `rules` is an array (OR logic). No nested objects keyed by field names.
- Omit unused change actions to reduce noise (canonical display order: new, changed, removed, existing).
- Multiple license policies can target distinct categories / severities.

## Common Rule Patterns
### Block Reciprocal / Copyleft
```json
{"rules": [
  {"field": "Category", "type": "eq", "value": "Reciprocal"},
  {"field": "Category", "type": "eq", "value": "Forbidden"}
]}
```
### Observe High Risk / Unknown
```json
{"rules": [
  {"field": "Severity", "type": "eq", "value": "HIGH"},
  {"field": "Severity", "type": "eq", "value": "CRITICAL"},
  {"field": "Severity", "type": "eq", "value": "UNKNOWN"}
]}
```
### Narrow to Package Patterns
```json
{"rules": [
  {"field": "PkgName", "type": "hasPrefix", "value": "@internal/"},
  {"field": "Name", "type": "contains", "value": "GPL"}
]}
```

## Output Strategy
| Goal | Pattern |
|------|---------|
| PR clarity | Table with Name, Category, Severity, PkgName for new/changed |
| Backlog visibility | Issue / file destination with only `existing` |
| Automation | JSON combined outputs for dashboards (see [Combining & Grouping](../output/combining-grouping.md)) |
| Minimal noise | Exclude `removed` unless tracking cleanup KPI |

Combined JSON semantics & grouping rules: [Combining & Grouping](../output/combining-grouping.md) (single concatenated array per destination).

## Exceptions / Allowlisting
Prefer inequality to exclude a single accepted license while still blocking class:
```json
"rules": [
  {"field":"Name","type":"contains","value":"GPL"},
  {"field":"Name","type":"ne","value":"GPL-2.0-with-classpath-exception"}
]
```
Document rationale via output `comment` or change description.

## Common Mistakes & Fixes
| Problem | Cause | Fix |
|---------|-------|-----|
| Rules ignored | Used legacy nested object (e.g. `{ "rules": { "severity": [...] }}`) | Convert to array of rule objects |
| Everything marked new | Missing main baseline (`/main`) or wrong `CI_EVENT` | Provide `/main` mount + set `CI_EVENT=pr` |
| Relationship filter no effect | `dependency_tree` disabled | Enable `global.dependency_tree=true` |
| Mixed JSON + markdown combine error | Combined outputs mix formats | Keep JSON groups pure or separate destinations |
| Severity not displayed | Omitted from `fields` | Add `Severity` to `fields` list |

## Best Practices
| Goal | Recommendation |
|------|---------------|
| Progressive rollout | Start by blocking clearly prohibited (GPL/AGPL) only; expand after confidence |
| Reduce review friction | Show only `new` + `changed` in PR; route `existing` to issue/file |
| Explain decisions | Include `Category` + `Severity` in blocking outputs |
| Maintainable config | Multiple narrow policies instead of one large rule list |
| Automation | Produce a combined JSON export for compliance dashboards |

## Related Topics
- [Policy System](../concepts/policy-system.md)
- [Diff-Based Analysis](../concepts/diff-analysis.md)
- [Progressive Enforcement](../operations/progressive-enforcement.md)
- [Combining & Grouping](../output/combining-grouping.md)
- [Configuration Overview](../configuration/overview.md)
- [Main Config Reference](../configuration/main-config.md)
- [Output Formats](../output/formats.md)

---
Next: manage dependency inventory with [Package Policies](./package.md).
