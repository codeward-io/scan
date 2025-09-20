# Main Config Reference (`config.json`)

Authoritative reference for the versioned Codeward policy file. For an orientation guide see: [Configuration Overview](./overview.md). For deeper field/operator definitions see: [Policy System](../concepts/policy-system.md). Style conventions: [Style & Naming Guide](./style-naming-guide.md).

Canonical change category order: `new, changed, removed, existing`.

## Purpose
Define what Codeward scans, how findings are filtered, which change categories trigger actions (`new | changed | removed | existing`), and how results are output. Runtime context (diff mounts, GitHub metadata, environment variables) is intentionally external.

## Top-Level Shape
```json
{
  "global": { "dependency_tree": false },
  "vulnerability": [ /* vulnerability policies */ ],
  "license": [ /* license policies */ ],
  "package": [ /* package policies */ ],
  "validation": [ /* validation policies */ ]
}
```
All arrays optional. Empty arrays are allowed for clarity.

### Global
| Key | Type | Effect |
|-----|------|--------|
| dependency_tree | boolean | Adds relationship enrichment fields (`Relationship`, `Parents`, `Children`, `Targets`) for package / license / vulnerability domains (slight performance cost). |

---
## Common Policy Schema (non‑Validation)
```json
{
  "name": "crit-high-block",
  "disabled": false,
  "actions": {"new": "block", "existing": "warn", "removed": "info", "changed": "warn"},
  "rules": [
    {"field": "Severity", "type": "eq", "value": "CRITICAL"},
    {"field": "Severity", "type": "eq", "value": "HIGH"}
  ],
  "outputs": [
    {"format": "markdown", "template": "table", "destination": "git:pr", "fields": ["VulnerabilityID","PkgName","Severity","FixedVersion"], "changes": ["new","existing"], "collapse": true},
    {"format": "json", "destination": "file:/results/vuln-diff.json", "changes": ["new","removed","changed"]}
  ]
}
```
Key Points:
- `rules` is ALWAYS an array of objects: `{"field": string, "type": operator, "value": string|number}`.
- Array entries are ORed (any rule passing keeps the record). Absence of `rules` means no filtering (all findings for the policy domain).
- Only include actions you need (omitted change type defaults to no action → treated as informational if surfaced by outputs).
- Multiple policies in the same domain can target different severities / subsets for progressive rollout.

### Allowed Record Fields (Summary)
Full canonical tables: [Policy System](../concepts/policy-system.md#allowed-record-fields-filter--display).
| Domain | Common Filter / Display Fields (subset) |
|--------|-----------------------------------------|
| vulnerability | VulnerabilityID, PkgName, PkgID, Severity, InstalledVersion, FixedVersion, Status |
| license | Name, Category, Severity, PkgName |
| package | Name, Version, Relationship, Parents, Children |
| validation | Key, Type, Value, Reason, Passing, Path |

---
## Validation Policy Differences
Validation requires content or filesystem context.

Additional Required Keys:
| Key | Purpose |
|-----|---------|
| type | One of `text|json|yaml|yml|filesystem` |
| path | File / directory / glob root for evaluation |

Example (filesystem existence):
```json
{
  "name": "require-security-md",
  "type": "filesystem",
  "path": ".",
  "actions": {"new": "block"},
  "rules": [ {"type": "exists", "path": "SECURITY.md"} ],
  "outputs": [ {"format": "markdown", "template": "table", "destination": "git:pr", "title": "Required Governance Files"} ]
}
```
Example (text content):
```json
{
  "name": "dockerfile-nonroot-user",
  "type": "text",
  "path": "**/Dockerfile",
  "actions": {"new": "block", "existing": "warn"},
  "rules": [
    {"type": "contains", "value": "USER"},
    {"type": "not_contains", "value": "root"}
  ],
  "outputs": [ {"format": "markdown", "template": "table", "destination": "git:pr", "fields": ["key","type","value","reason","passing","path"]} ]
}
```
Validation rule object shapes differ (some use `path` instead of `field/value`). See: [Policy System — Validation Rule Shapes](../concepts/policy-system.md#validation-policy-rule-shapes).

---
## Domain Examples
### Vulnerability (Progressive Rollout Pair)
```json
[
  {
    "name": "crit-block",
    "actions": {"new": "block", "existing": "warn"},
    "rules": [ {"field": "Severity", "type": "eq", "value": "CRITICAL"} ],
    "outputs": [ {"format": "markdown", "template": "table", "destination": "git:pr", "fields": ["VulnerabilityID","PkgName","Severity","FixedVersion"], "changes": ["new","existing"], "collapse": true} ]
  },
  {
    "name": "high-observe",
    "actions": {"new": "warn"},
    "rules": [ {"field": "Severity", "type": "eq", "value": "HIGH"} ],
    "outputs": [ {"format": "json", "destination": "file:/results/highs.json", "changes": ["new"]} ]
  }
]
```
### License
```json
[
  {
    "name": "prohibited-gpl",
    "actions": {"new": "block", "existing": "warn"},
    "rules": [ {"field": "Name", "type": "contains", "value": "GPL"} ],
    "outputs": [ {"format": "markdown", "template": "table", "destination": "git:pr", "fields": ["Name","Category","Severity","PkgName"], "changes": ["new","changed"]} ]
  }
]
```
### Package
```json
[
  {
    "name": "removed-deps-warn",
    "actions": {"removed": "warn"},
    "rules": [ {"field": "Name", "type": "hasPrefix", "value": "@types/"} ],
    "outputs": [ {"format": "markdown", "template": "table", "destination": "log:stdout", "fields": ["Name","Version","Relationship"], "changes": ["removed"]} ]
  }
]
```
### Validation (text) — shown earlier

---
## Rule Operators (Summary)
| Operator | Meaning |
|----------|---------|
| eq / ne | Equality / inequality |
| lt / gt / le / ge | Numeric comparisons (where applicable) |
| contains / not_contains | Substring match |
| hasPrefix / hasSuffix | String edge match |
| regex | Regular expression (RE2 compatible) |
| exists / not_exists | Presence (validation / filesystem) |

Refer to [Policy System](../concepts/policy-system.md#rule-operators) for authoritative list and domain applicability.

---
## Outputs (Quick Reference)
Minimal per-policy output object:
```json
{"format": "markdown", "template": "table", "destination": "git:pr", "fields": ["VulnerabilityID","Severity"], "changes": ["new"]}
```
Common keys: `format`, `template` (omit for JSON), `destination`, `fields`, `changes`, `group_by`, `combined`, `collapse`, `title`, `comment`.

Detailed formatting / destinations: [Output Formats](../output/formats.md), [Output Destinations](../output/destinations.md).

Combined JSON outputs must all be JSON and share a destination (enforced). Markdown/HTML may share a comment thread when combined.

Combined JSON semantics (anywhere in this file): a single concatenated array per destination (no wrapper object) when `combined:true` outputs share format+destination.

---
## Actions
| Action | Effect | Typical Use |
|--------|--------|-------------|
| info | Record only (non-blocking) | Baseline visibility |
| warn | Non-blocking signal (non-zero severity) | Progressive rollout step |
| block | Fails run (non-zero exit) | Critical guardrail |

Progressive enforcement pattern: start with `block` only for CRITICAL; later add HIGH; then gradually include other categories as backlog shrinks.

---
## AI Governance Rationale
AI‑assisted commits can introduce high-severity vulnerabilities or prohibited licenses at scale. Diff‑aware policies let you block only net‑new critical risk while still surfacing historical issues for phased remediation, maintaining developer velocity without silent regression.

---
## Common Mistakes & Fixes
| Problem | Cause | Fix |
|---------|-------|-----|
| Rules ignored | Used legacy nested object (e.g. `"rules": { "severity": [...] }`) | Replace with array of `{"field":"Severity","type":"eq","value":"CRITICAL"}` entries |
| Everything marked new | Missing main branch mount or `CI_EVENT=pr` | Provide `/main`, `/branch` volumes + set `CI_EVENT=pr` |
| Block not triggered | Only `existing` findings present | Ensure actions include `new` and thresholds capture net‑new items |
| Validation rule error | Missing `type` / `path` for validation policy | Include both required keys |
| Mixed JSON + markdown in combined output | Incompatible formats with `combined:true` | Use uniform format or separate outputs |
| Unsupported field in `fields` | Field not in allowed domain set | Check allowed field tables; remove or adapt |

---
## Best Practices
| Goal | Recommendation |
|------|---------------|
| Noise reduction | Limit PR outputs to `new` + `changed` changes; send `existing` to file/issue |
| Explainability | Include fields that justify the action (e.g. `Severity`, `FixedVersion`) |
| Progressive rollout | Use multiple policies per domain for staged enforcement |
| Observability | At least one JSON output per critical policy for dashboards |
| Maintainability | Small focused policies > one large multi-purpose policy |
| Reusability | Keep environment-dependent values (paths, tokens) out of config file |

---
## Related & Next Steps
- [Policy System](../concepts/policy-system.md)
- [Diff-Based Analysis](../concepts/diff-analysis.md)
- [Output Formats](../output/formats.md)
- [Output Destinations](../output/destinations.md)
- [Configuration Overview](./overview.md)

---
Minimal final reminder: use array-based `rules` everywhere (except specialized validation rule forms) and keep runtime context in environment variables, not the config.
