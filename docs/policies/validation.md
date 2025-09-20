# Validation Policies

Validation policies assert security, compliance, and project hygiene rules against file contents (text, json, yaml), filesystem structure, or configuration values. They complement vulnerability / license / package policies by covering custom guardrails your org needs. PR outputs emphasize new/changed violations; baseline scans can surface the full existing set for planned remediation.

## Overview
Each validation policy evaluates one path pattern (file(s) or filesystem) using rule objects. Diff categories (`new | changed | removed | existing`) still drive actions (see semantics: [Diff-Based Analysis](../concepts/diff-analysis.md)), but validation policies focus on pass/fail of rule checks within the current head version. Use them to block introduction of insecure patterns while observing existing debt. Staged rollout patterns: [Progressive Enforcement](../operations/progressive-enforcement.md).

> Style & naming conventions (actions formatting `info | warn | block`, canonical change order) live in the [Style & Naming Guide](../configuration/style-naming-guide.md).

Canonical change category order everywhere: new, changed, removed, existing.

## Rationale / Principles
| Principle | Why |
|-----------|-----|
| Explicit assertions | Prevent silent drift in critical config (Dockerfile, CI workflows, package.json). |
| Diff gating | Only new / changed violations block; existing backlog can surface but remain non‑blocking. |
| Narrow policies | Single intent per policy simplifies tuning & rollout. |
| Progressive enforcement | Start warn → graduate to block after confidence. |
| AI governance | AI‑generated config/scripts may omit mandatory hardening; policies catch omissions early. |

## Schema (Subset)
Full schema, operators, allowed fields: [Policy System](../concepts/policy-system.md). Style conventions: [Style & Naming Guide](../configuration/style-naming-guide.md).
```json
{
  "validation": [
    {
      "name": "dockerfile-non-root",
      "type": "text",
      "path": "Dockerfile",
      "actions": {"new": "block", "changed": "block", "existing": "warn"},
      "rules": [ {"type": "contains", "value": "USER"} ],
      "outputs": [ {"format": "markdown", "template": "table", "destination": "git:pr", "fields": ["key","reason","passing"], "changes": ["new","changed","existing"]} ]
    }
  ]
}
```
Notes:
* Same per‑change `actions` map as other policies (no single `action` key).
* `rules` array (OR). No nested keyed objects.
* Types: text | json | yaml | yml | filesystem.
* Filesystem rules use `path` with `exists` / `not_exists`.

## Rule Reference (Implemented)
| Type Context | Keys | Operators |
|--------------|------|-----------|
| text | type, value | regex, contains, not_contains, hasPrefix, hasSuffix, eq, ne |
| json / yaml / yml | type, key, value | eq, ne, lt, gt, le, ge, contains, not_contains, hasPrefix, hasSuffix, regex, exists, not_exists |
| filesystem | type, path | exists, not_exists |

Examples:
```json
{"rules":[{"type":"regex","value":"^FROM.+@sha256"}]}
{"rules":[{"type":"exists","key":"scripts.test"}]}
{"rules":[{"type":"not_contains","value":"password"}]}
{"rules":[{"type":"exists","path":"README.md"}]}
```

## Progressive Enforcement Examples
(See strategy guidance: [Progressive Enforcement](../operations/progressive-enforcement.md))
### Harden Dockerfile Stepwise
```json
"validation": [
  {"name":"dockerfile-user","type":"text","path":"Dockerfile","actions":{"new":"warn","changed":"warn"},
   "rules":[{"type":"contains","value":"USER"}],
   "outputs":[{"format":"markdown","template":"table","destination":"git:pr","fields":["key","reason","passing"],"changes":["new","changed"]}]},
  {"name":"dockerfile-user-block","type":"text","path":"Dockerfile","actions":{"new":"block","changed":"block","existing":"warn"},
   "rules":[{"type":"contains","value":"USER"}],
   "outputs":[{"format":"markdown","template":"table","destination":"git:pr","fields":["key","reason","passing"],"changes":["new","changed","existing"],"collapse":true}]}
]
```
### Secure CI Workflow
```json
{"name":"workflow-permissions","type":"yaml","path":".github/workflows/*.yml",
 "actions":{"new":"block","changed":"block","existing":"warn"},
 "rules":[
   {"type":"eq","key":"permissions.contents","value":"read"},
   {"type":"not_contains","key":"jobs.*.steps[*].run","value":"curl | sh"}
 ],
 "outputs":[{"format":"markdown","template":"table","destination":"git:pr","fields":["key","reason","passing"],"changes":["new","changed"]}]
}
```
### Required Files (filesystem)
```json
{"name":"required-readme","type":"filesystem","path":".",
 "actions":{"new":"info","changed":"info","removed":"info","existing":"info"},
 "rules":[{"type":"exists","path":"README.md"}],
 "outputs":[{"format":"json","destination":"file:/results/required-files.json","combined":true}]}
```

## Output Strategy
| Goal | Pattern |
|------|---------|
| PR remediation clarity | Markdown table (key, reason, passing) with only failing new/changed (filter rules + changes). |
| Backlog surfacing | Separate policy routing `existing` to file:/ json. |
| Automation ingestion | Combined JSON export; omit template field (see [Combining & Grouping](../output/combining-grouping.md)). |
| Minimal noise | Restrict fields to key, reason, passing. |

Combined JSON semantics: a single concatenated array containing all selected records across all matching policies (not multiple JSON roots).

## Best Practices
| Objective | Recommendation |
|-----------|---------------|
| Avoid over-blocking | Start with warnings to tune patterns. |
| Express rationale | Use `comment` for why rule matters. |
| Keep rules tight | Prefer explicit regex anchors to broad contains. |
| Separate intents | One policy per domain (Dockerfile user, workflow perms). |
| Maintain consistency | Reuse shared output destinations & formats. |

## Common Mistakes & Fixes
| Problem | Cause | Fix |
|---------|-------|-----|
| Single `action` key used | Legacy schema misunderstanding | Replace with actions map (`new`,`changed`,`removed`,`existing`). |
| Rules ignored | Nested object form; missing array | Use `"rules": [ {..}, {..} ]`. |
| JSON output error (template) | Template set while format=json | Remove `template` key for json outputs. |
| Fields rejected | Unsupported display field | Use only key,type,value,reason,passing,path. |
| Over-reporting existing debt | Included `existing` in PR output | Limit `changes` to new/changed for PR outputs. |

## Related / Next Steps
* Core schema details: [Policy System](../concepts/policy-system.md)
* Diff semantics: [Diff-Based Analysis](../concepts/diff-analysis.md)
* Rollout strategies: [Progressive Enforcement](../operations/progressive-enforcement.md)
* Output usage: [Output Formats](../output/formats.md) & [Combining & Grouping](../output/combining-grouping.md)
* Starter patterns: [Starter Configurations](../examples/starter-configs.md)

---
AI Governance: Validation policies ensure AI or human rapid edits cannot remove essential security lines (e.g., non-root USER) or add unsafe constructs without immediate detection.
