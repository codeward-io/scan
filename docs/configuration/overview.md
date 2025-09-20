# Configuration Overview

Codeward configuration turns governance into code: a single JSON file (`config.json`) declaring policies (vulnerability, license, package, validation), actions per change category (`new | changed | removed | existing` – see [Glossary](../concepts/glossary.md#change-categories)), and outputs. Runtime context (repository mounts, event type, GitHub metadata) is supplied via environment variables—kept intentionally outside the versioned policy file.

## Design Goals
- **Diff‑Aware**: Differentiate net‑new risk from historical backlog.
- **Policy‑First**: All enforcement & reporting flows through explicit, reviewable policy objects.
- **Deterministic Outputs**: Same inputs + config → stable Markdown/HTML and JSON arrays (safe for automation).
- **Low Friction**: Only commit what you intend to govern; no absolute host paths or tokens in `config.json`.

## Top-Level Structure
```json
{
  "global": { "dependency_tree": false },
  "vulnerability": [ /* vulnerability policy objects */ ],
  "license": [ /* license policy objects */ ],
  "package": [ /* package policy objects */ ],
  "validation": [ /* validation policy objects */ ]
}
```
All arrays may be empty. Omit an array entirely if that domain is not used (keeping empty arrays improves clarity for reviewers). Style conventions: [Style & Naming Guide](./style-naming-guide.md).

### Global Settings
| Key | Type | Effect |
|-----|------|--------|
| `dependency_tree` | boolean | Adds relationship enrichment fields (`Relationship`, `Parents`, `Children`, `Targets`) (slight performance cost). |

## Policy Object (Non‑Validation Types)
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
    {"format": "json", "destination": "file:/results/vuln-changes.json", "changes": ["new","changed","removed"], "combined": true}
  ]
}
```
Key points:
- `rules` is always an array of objects `{field,type,value}` (OR logic). Never use nested objects keyed by field name.
- Omit unused change keys in `actions` to reduce noise (canonical order: new, changed, removed, existing).

## Validation Policy Differences
Validation adds required keys:
| Key | Purpose |
|-----|---------|
| `type` | One of `text|json|yaml|yml|filesystem` |
| `path` | Target file / glob / directory |
| `rules` | Validation rule objects (shape depends on `type`) |

Example (filesystem):
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
See: [Policy System](../concepts/policy-system.md#validation-policy-rule-shapes).

## Actions (Per Change Category)
| Action | Effect | Typical Use |
|--------|--------|------------|
| `info` | Record only | Baseline visibility |
| `warn` | Non‑blocking signal | Socialize standards |
| `block` | Fails run (non‑zero exit) | Enforce critical guardrails |

Progressive adoption guidance: [Progressive Enforcement](../operations/progressive-enforcement.md).

## Allowed Fields
Display / filter field reference lives in [Policy System](../concepts/policy-system.md#allowed-record-fields-filter--display). Use only those field names in `rules` and `fields` lists.

## Output Configuration (Per Policy)
```json
{
  "format": "markdown|html|json",
  "destination": "git:pr|git:issue|file:/path|log:stdout|log:stderr",
  "template": "table|text",        // empty when format=json
  "fields": ["FieldA","FieldB"],  // subset of allowed fields
  "group_by": ["Field"],            // optional grouping
  "changes": ["new","changed"],   // category filter
  "combined": true,                  // merge with other outputs (same dest+format)
  "collapse": true,                  // markdown/html collapsible section
  "title": "Custom Title",
  "comment": "Context line beneath title"
}
```
Combined semantics & grouping: [Combining & Grouping](../output/combining-grouping.md).

## Environment & Runtime (Outside `config.json`)
| Variable | Purpose |
|----------|---------|
| `CI_EVENT` | `pr` enables diff (needs `/main` & `/branch`); `main` single state |
| `GITHUB_TOKEN`, `GITHUB_OWNER`, `GITHUB_REPO`, `GITHUB_PR_NR` | Required for git destinations (context dependent) |
| `CONFIG_PATH` | Override config file path |
| `PRIVATE_CONFIG_PATH` | Optional private override file |
| `CI` | Signals CI context (affects template path resolution) |

Mounts (`/main`, `/branch`, `/results`, `/.cache`) supplied by the Action / Docker.

## AI Governance Rationale
Centralizing rules in a diff‑aware config allows rapid, explainable enforcement against AI‑generated or bulk refactors: net‑new critical vulnerabilities or prohibited licenses can block immediately while legacy issues remain visible but non‑blocking until phased enforcement escalates.

## Common Mistakes & Fixes
| Problem | Cause | Fix |
|---------|-------|-----|
| Rules ignored | Used object mapping (e.g., `{ "severity": [...] }`) | Use array of `{ "field": "Severity", ...}` objects |
| Mixed formats in combined output | `combined:true` across different formats | Keep combined destination uniform (all JSON or all markdown/html) |
| Empty PR comment | Only `existing` changes configured | Include `new` / `changed` for PR destinations |
| Everything labeled new | Missing `/main` mount or `CI_EVENT=pr` | Provide both volumes + event env var |
| Validation policy fails | Missing `type` or `path` | Add required keys |

## Best Practices
| Goal | Recommendation |
|------|---------------|
| Minimize noise | Limit PR outputs to `new` + `changed`; send `existing` elsewhere |
| Justify blocking | Include `Severity`, `FixedVersion`, or `Category` fields in blocking outputs |
| Progressive rollout | Start by blocking critical; expand to high after tuning |
| Observability | Add at least one JSON output per critical policy |
| Maintainability | Use multiple focused policies instead of a monolith |

## Auto-Applied Default (When `config.json` Absent) {#default-config}
If no `.codeward/config.json` is present, Codeward applies a built-in default configuration so first‑run scans still produce actionable output. This default is intentionally opinionated (it blocks high/critical — and medium for vulnerabilities — immediately). Customize with your own `config.json` if you prefer an initial non‑blocking rollout.

Default (verbatim) snippet:
```json
{
  "global": { "dependency_tree": false },
  "vulnerability": [
    {
      "name": "Vulnerability issues",
      "actions": { "new": "block", "existing": "warn", "removed": "info" },
      "rules": { "severity": [
        { "type": "eq", "value": "CRITICAL" },
        { "type": "eq", "value": "HIGH" },
        { "type": "eq", "value": "MEDIUM" }
      ]},
      "outputs": [
        { "collapse": true, "format": "markdown", "fields": ["PkgID","VulnerabilityID","InstalledVersion","FixedVersion","Severity"], "destination": "git:pr", "title": "[codeward.io] Vulnerability Issues", "comment": "Please contact the security team to review the vulnerability issues.", "changes": ["new","removed","existing"], "template": "table" },
        { "collapse": true, "format": "markdown", "fields": ["PkgID","VulnerabilityID","InstalledVersion","FixedVersion","Severity"], "destination": "git:issue", "title": "[codeward.io] Vulnerability Issues", "comment": "Please contact the security team to review the vulnerability issues.", "changes": ["existing"], "template": "table" }
      ],
      "dependency_tree": false
    }
  ],
  "license": [
    {
      "name": "License issues",
      "actions": { "new": "block", "existing": "warn", "removed": "info" },
      "rules": { "severity": [
        { "type": "eq", "value": "UNKNOWN" },
        { "type": "eq", "value": "CRITICAL" },
        { "type": "eq", "value": "HIGH" }
      ]},
      "outputs": [
        { "format": "markdown", "collapse": true, "fields": ["Severity","PkgName","Category","Name"], "destination": "git:pr", "changes": ["existing","new","removed"], "comment": "Please contact the legal team to review the license issues.", "template": "table" },
        { "format": "markdown", "collapse": true, "fields": ["Severity","PkgName","Category","Name"], "destination": "git:issue", "changes": ["existing"], "comment": "Please contact the legal team to review the license issues.", "template": "table" }
      ]
    }
  ],
  "package": [],
  "validation": []
}
```
Legacy rule shape note: The default uses an older nested `"rules": { "severity": [...] }` structure. When authoring your own config use the canonical array form:
```json
"rules": [
  { "field": "Severity", "type": "eq", "value": "CRITICAL" },
  { "field": "Severity", "type": "eq", "value": "HIGH" }
]
```
To start in observe-only mode instead of blocking, create a minimal config overriding `actions` (e.g. `"new": "warn"`) as shown below.

## Example Minimal Config
```json
{
  "vulnerability": [
    {
      "name": "crit-block",
      "actions": {"new": "block", "existing": "warn"},
      "rules": [ {"field": "Severity", "type": "eq", "value": "CRITICAL"} ],
      "outputs": [ {"format": "markdown", "template": "table", "destination": "git:pr", "fields": ["VulnerabilityID","PkgName","Severity","FixedVersion"], "changes": ["new","existing"]} ]
    }
  ],
  "license": [],
  "package": [],
  "validation": []
}
```

## Related Topics
- [Policy System](../concepts/policy-system.md)
- [Diff-Based Analysis](../concepts/diff-analysis.md)
- [Main Config Reference](./main-config.md)
- [Output Formats](../output/formats.md)
- [Combining & Grouping](../output/combining-grouping.md)

---
Next: dive into field/operator details and advanced patterns in the [Main Config Reference](./main-config.md).
