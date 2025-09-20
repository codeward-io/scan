# Scanning Types

Codeward unifies four scan domains under one **diff + policy** model. Each produces structured records classified as `new`, `changed`, `removed`, or `existing` (see [Diff-Based Analysis](./diff-analysis.md) and [Glossary](./glossary.md)). Policies then apply `info | warn | block` actions per category (see [Glossary](./glossary.md#actions)) and route findings to outputs. Style conventions: [Style & Naming Guide](../configuration/style-naming-guide.md).

## Domains Overview
| Domain | Source | Surfaces | Typical Governance Focus |
|--------|--------|----------|--------------------------|
| Vulnerability | Trivy | CVE records per package (severity, fix version) | Block introduction of critical/high risk; track remediation |
| License | Trivy | License name + category + severity mapping | Prevent prohibited categories (e.g., Copyleft) |
| Package | Trivy | Dependency presence & version deltas | Review new additions; observe version shifts |
| Validation | Filesystem / content | File existence & content rule results | Enforce structural & security conventions |

> All share the same change categorization logic (no per-domain variations). Avoid redefining categories here—central source of truth is the Diff-Based Analysis page.

## 1. Vulnerability
Highlights security risk in dependencies.

Key fields (filter / display): `VulnerabilityID`, `PkgID`, `PkgName`, `InstalledVersion`, `FixedVersion`, `Status`, `Severity`, `Relationship`, `Children`, `Parents`, `Targets`.

Example policy (critical block, high observe):
```json
{
  "vulnerability": [
    {
      "name": "crit-high",
      "actions": {"new": "block", "existing": "warn"},
      "rules": [
        {"field": "Severity", "type": "eq", "value": "CRITICAL"},
        {"field": "Severity", "type": "eq", "value": "HIGH"}
      ],
      "outputs": [
        {"format": "markdown", "template": "table", "destination": "git:pr", "fields": ["VulnerabilityID","PkgName","Severity","FixedVersion"], "changes": ["new"], "collapse": true}
      ]
    }
  ]
}
```
Governance focus: Gate net-new high severity risk while surfacing backlog separately (`existing` in non‑PR destination if desired).

## 2. License
Tracks license posture and category risk.

Key fields: `Severity`, `Category`, `PkgName`, `Name`, `Relationship`, `Children`, `Parents`, `Targets`.

Example (block Copyleft introductions):
```json
{
  "license": [
    {
      "name": "no-copyleft-new",
      "actions": {"new": "block", "existing": "warn"},
      "rules": [ {"field": "Category", "type": "eq", "value": "Copyleft"} ],
      "outputs": [
        {"format": "markdown", "template": "text", "destination": "git:pr", "title": "License Category Introductions", "changes": ["new"]}
      ]
    }
  ]
}
```
Governance focus: Prevent prohibited license categories from entering via large or AI‑generated change sets.

## 3. Package
Provides dependency inventory diffs (additions, removals, version changes).

Key fields: `ID`, `Name`, `Version`, `Relationship`, `Children`, `Parents`, `Targets`.

Example (observe new direct dependencies):
```json
{
  "package": [
    {
      "name": "new-direct",
      "actions": {"new": "warn"},
      "rules": [ {"field": "Relationship", "type": "eq", "value": "direct"} ],
      "outputs": [ {"format": "json", "destination": "file:/results/new-direct.json", "changes": ["new"], "fields": ["Name","Version","Relationship"]} ]
    }
  ]
}
```
Governance focus: Review newly introduced packages before merge; optionally escalate to block later.

## 4. Validation (Custom Rules)
Enforces structural & content expectations across files / trees.

Requirements: each validation policy includes `type` (`text|json|yaml|yml|filesystem`) & `path`. Rule shapes vary—see [Policy System](./policy-system.md#validation-policy-rule-shapes).

Example (require SECURITY.md present):
```json
{
  "validation": [
    {
      "name": "require-security-md",
      "type": "filesystem",
      "path": ".",
      "actions": {"new": "block"},
      "rules": [ {"type": "exists", "path": "SECURITY.md"} ],
      "outputs": [ {"format": "markdown", "template": "table", "destination": "git:pr", "title": "Required Governance Files"} ]
    }
  ]
}
```
Governance focus: Guarantee required governance artifacts and disallow insecure patterns (e.g., unsafe CI steps, inline secrets).

## Unified Rule Model (Non-Validation)
All vulnerability, license, and package policies use filter rule objects:
```
{"field":"<Field>","type":"<operator>","value":"<string>"}
```
Multiple rules are ORed. Use separate policies when intents differ (improves output clarity). Operator list & allowed fields: see [Policy System](./policy-system.md#allowed-record-fields-filter--display).

## JSON Output
All JSON outputs are arrays. Combined JSON destinations yield a single concatenated array (no wrapper object). See [Combining & Grouping](../output/combining-grouping.md) for rules & constraints.

## Best Practices
| Goal | Recommendation |
|------|----------------|
| Minimize PR noise | Restrict PR outputs to `new` (and `changed` where risk shifts) |
| Encourage remediation | Provide a separate output highlighting only `removed` |
| Progressive enforcement | Start with blocking only critical (or Copyleft) introductions |
| Deterministic automation | Emit one combined JSON artifact for dashboards |
| Clarity | Small focused policies rather than large rule sets |

## Common Mistakes & Fixes
| Symptom | Cause | Fix |
|---------|-------|-----|
| Rule schema error | Used legacy nested object | Convert to array of rule objects |
| All items new | Missing `/main` mount or `CI_EVENT=pr` | Mount baseline & set env var |
| Empty JSON | Rules filtered everything or `changes` too narrow | Broaden or relax filters temporarily |
| Mixed formats in combined JSON | Combined group included markdown output | Restrict group to JSON outputs only |
| Invalid field (e.g., license) | Non-existent field name | Use canonical field names (see Policy System) |

## Related Topics
- [Diff-Based Analysis](./diff-analysis.md)
- [Policy System](./policy-system.md)
- [Combining & Grouping](../output/combining-grouping.md)
- [Output Formats](../output/formats.md)

---
Next: author policies — see the [Policy System](./policy-system.md).
