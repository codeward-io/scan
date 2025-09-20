---
id: policy-system
title: Policy System
description: Core diff-aware policy model: actions per change category, rules, outputs, combining, and validation extensions.
keywords:
  - policy system
  - diff-aware
  - governance
  - security
  - configuration
---
<!-- filepath: /Users/tambet/Documents/GitHub/codeward-io/docs/docs/concepts/policy-system.md -->
# Policy System

Codeward applies a consistent, diff‑aware policy model across all scan domains so you can codify governance (security, license, supply chain, structural validation) and enforce it pre‑merge.

Canonical change category order: `new, changed, removed, existing` (see [Glossary](./glossary.md)). Formatting & naming conventions: [Style & Naming Guide](../configuration/style-naming-guide.md).

## Why Policies (AI & Velocity Context)
High‑velocity and AI‑assisted changes can silently introduce vulnerable or prohibited components. Policies:
- Express guardrails as code (repeatable, reviewable)
- Reduce subjective review effort (objective actions per change type)
- Allow gradual tightening (warn → block) without large rewrites

## Supported Policy Types
| Type | Source Data | Primary Use |
|------|-------------|-------------|
| vulnerability | Trivy | Govern CVE exposure (severity / fixed versions) |
| license | Trivy | Restrict license categories / names (compliance posture) |
| package | Trivy | Track dependency additions / removals / version shifts |
| validation | Filesystem / content | Assert structural & content requirements (text/json/yaml/filesystem) |

## Change Categories (Diff Mode)
`new`, `changed`, `removed`, `existing` — canonical order (see [Glossary](./glossary.md)). Each policy defines an action for any subset (omitted keys ignored).

## Actions {#actions}
| Action | Effect |
|--------|--------|
| info | Record in output only |
| warn | Record + non‑blocking signal |
| block | Marks run as failing (non‑zero exit) if any section produced |

A policy need only define the change keys it cares about (others can be omitted).

## Configuration Overview
Top-level arrays in `config.json`: `vulnerability`, `license`, `package`, `validation` (any may be empty). Each element:
```
{
  "name": "string",
  "disabled": false,          // optional
  "actions": {"new":"block","existing":"warn"},
  "rules": [ {"field":"Severity","type":"eq","value":"CRITICAL"} ],
  "outputs": [ { ...output config... } ],
  ...validation only extras...
}
```
Validation policies add required keys: `type` (text|json|yaml|yml|filesystem) and `path` (target file / directory). Rule objects differ per validation type (see below).

### Filter Rule Object (non-validation policies)
```
{"field": "<AllowedField>", "type": "op", "value": "string"}
```
Allowed operators (generic): `eq`, `ne`, `lt`, `gt`, `le`, `ge`, `contains`, `not_contains`, `hasPrefix`, `hasSuffix`, `regex` (depending on field semantics). Only fields from the underlying record type are valid.

### Allowed Record Fields (Filter / Display) (Canonical)
| Policy | Fields |
|--------|--------|
| vulnerability | VulnerabilityID, PkgID, PkgName, InstalledVersion, FixedVersion, Status, Severity, Relationship, Children, Parents, Targets |
| license | Severity, Category, PkgName, Name, Relationship, Children, Parents, Targets |
| package | ID, Name, Version, Relationship, Children, Parents, Targets |
| validation | Key, Type, Value, Reason, Passing, Path (display fields; filtering not applied in same manner) |

## Rule Operators (Canonical) {#rule-operators}
Generic operators (filtering across vulnerability, license, package domains): `eq`, `ne`, `lt`, `gt`, `le`, `ge`, `contains`, `not_contains`, `hasPrefix`, `hasSuffix`, `regex`.

Validation adds (by rule type):
- Text / JSON / YAML / YML: same generic set above, plus context-specific usage of values.
- JSON / YAML / YML additionally: `exists`, `not_exists` (for key presence checks) if using path-style rules (see Validation shapes below).
- Filesystem: `exists`, `not_exists`.

Summary:
| Operator | Meaning |
|----------|---------|
| eq / ne | Equality / inequality |
| lt / gt / le / ge | Numeric comparison (where applicable) |
| contains / not_contains | Substring presence/absence |
| hasPrefix / hasSuffix | String edge match |
| regex | RE2-compatible regular expression |
| exists / not_exists | Presence/absence (validation & filesystem) |

(Operator applicability: see individual validation rule shape tables below.)

## Output Configuration
(See also: [Combining & Grouping](../output/combining-grouping.md) for destination merge rules.)
High-level description of output configuration options. Combined JSON groups produce a single concatenated array (no wrapper object; uniform format required).

### Per Policy Output
| Field | Purpose |
|-------|---------|
| format | markdown \| html \| json |
| destination | file:&lt;path&gt; \| log:stdout \| log:stderr \| git:pr \| git:issue |
| template | table \| text (ignored for json) |
| fields | Subset of allowed display fields |
| group_by | Array of fields to merge/group results |
| changes | Limit change categories for this output |
| combined | Combine multiple policy reports at same destination |
| collapse | (markdown/html) make section collapsible |
| title/comment | Presentation overrides |

JSON outputs must omit `template`. Combined JSON groups must be uniformly JSON.

## Validation Policy Rule Shapes
| Validation Type | Rule Keys | Rule Types |
|-----------------|----------|-----------|
| text | type, value | regex, contains, not_contains, hasPrefix, hasSuffix, eq, ne |
| json / yaml / yml | type, key, value | eq, ne, lt, gt, le, ge, contains, not_contains, hasPrefix, hasSuffix, regex, exists, not_exists |
| filesystem | type, path | exists, not_exists |

Example (text validation – disallow common secret tokens):
```json
{
  "validation": [
    {
      "name": "disallow-inline-secrets",
      "type": "text",
      "path": "app/config.env",        
      "actions": {"new": "block"},
      "rules": [
        {"type": "regex", "value": "(?i)(password|secret|apikey)\s*=\s*.+"}
      ],
      "outputs": [
        {"format": "markdown", "template": "text", "destination": "git:pr", "title": "Potential Inline Secrets"}
      ]
    }
  ]
}
```
(If targeting many files, invoke multiple validation policies or supply path patterns per implementation constraints.)

## Correct Example Policies
### Vulnerability (Block Critical & High New; Warn Existing)
_Example focuses on severity gating and mixed output formats._
```json
{
  "vulnerability": [
    {
      "name": "crit-high-vulns",
      "actions": {"new": "block", "existing": "warn", "removed": "info", "changed": "warn"},
      "rules": [
        {"field": "Severity", "type": "eq", "value": "CRITICAL"},
        {"field": "Severity", "type": "eq", "value": "HIGH"}
      ],
      "outputs": [
        {"format": "markdown", "template": "table", "destination": "git:pr", "fields": ["VulnerabilityID","PkgName","Severity","FixedVersion"], "changes": ["new","existing"], "collapse": true},
        {"format": "json", "destination": "file:/results/vuln-changes.json", "changes": ["new","removed","changed"]}
      ]
    }
  ]
}
```
### License (Block Introduction of Certain Categories)
```json
{
  "license": [
    {
      "name": "no-copyleft-intro",
      "actions": {"new": "block", "existing": "warn", "removed": "info"},
      "rules": [
        {"field": "Category", "type": "eq", "value": "Copyleft"}
      ],
      "outputs": [
        {"format": "markdown", "template": "text", "destination": "git:pr", "title": "License Category Introductions", "changes": ["new"]}
      ]
    }
  ]
}
```
### Package (Track Additions Only)
```json
{
  "package": [
    {
      "name": "new-deps-tracker",
      "actions": {"new": "warn"},
      "rules": [],
      "outputs": [
        {"format": "json", "destination": "file:/results/new-deps.json", "changes": ["new"], "fields": ["Name","Version","Relationship","Parents"]}
      ]
    }
  ]
}
```
### Validation (Filesystem Presence)
```json
{
  "validation": [
    {
      "name": "require-security-policy",
      "type": "filesystem",
      "path": ".",  
      "actions": {"new": "block"},
      "rules": [
        {"type": "exists", "path": "SECURITY.md"}
      ],
      "outputs": [
        {"format": "markdown", "template": "table", "destination": "git:pr", "title": "Required Governance Files"}
      ]
    }
  ]
}
```

## Multiple Policies Per Type
Combine granular policies instead of one large, hard‑to‑tune policy:
- Separate severity bands
- Separate license categories (copyleft vs restricted vs audit‑only)
- Separate package introduction tracker vs remediation tracker

## Progressive Enforcement
Moved to dedicated page: see [Progressive Enforcement](../operations/progressive-enforcement.md) for phased rollout strategy and escalation guidance.

## Best Practices
| Objective | Recommendation |
|-----------|---------------|
| Minimize noise | Limit PR outputs to `new` + `changed`; route `existing` elsewhere |
| Show improvements | Dedicated output with `removed` only |
| Justify blocking | Include remediation fields (e.g., `FixedVersion`) |
| Automate dashboards | Emit JSON arrays (deterministic) |
| Avoid config sprawl | Compose many small focused policies |
| Safe rollout | Apply phases from Progressive Enforcement page |

## Common Mistakes & Fixes
| Problem | Cause | Fix |
|---------|-------|-----|
| Validation policy rejected | Missing `type` or `path` | Add required keys |
| Rules ignored | Wrong schema (object instead of array) | Use an array of rule objects | 
| Combined JSON error | Mixed markdown + JSON in combined group | Use single format per combined destination |
| Everything blocked | Overbroad rule + block on all change types | Narrow rules / limit `new` to block |
| License rule ineffective | Used non-existent field name (e.g., "license") | Use `Name` or `Category` |

## Related Topics
- [Diff Analysis](./diff-analysis.md)
- [Output Formats](../output/formats.md)
- [Main Config Reference](../configuration/main-config.md)
- [Progressive Enforcement](../operations/progressive-enforcement.md)
- [Style & Naming Guide](../configuration/style-naming-guide.md)

---
Next: review full configuration structure in the [Main Config Reference](../configuration/main-config.md).
