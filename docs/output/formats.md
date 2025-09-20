---
title: Output Formats
description: Diff-aware policy output formats: markdown, HTML, and JSON arrays for deterministic automation.
---
# Output Formats

Authoritative reference for how policy outputs are rendered. Destinations (PR comment, issue, file, log) are covered separately: see [Output Destinations](./destinations.md).

## Overview
Codeward renders policy findings in one of three formats: `markdown`, `html`, `json`. Markdown/HTML use templates (`table` or `text`); JSON is always structured data (no template). Formats are per‑output (each policy may define multiple outputs). Canonical change category order: `new, changed, removed, existing`.

## Design Principles
- **Deterministic**: Same inputs + config → identical output (supports automation & diffing).
- **Diff‑Focused**: Change category filtering (`changes`) keeps PR noise low.
- **Composable**: Multiple policy outputs can be merged (see [Combining & Grouping](./combining-grouping.md)).
- **Automation Friendly**: JSON arrays for machine ingestion; markdown/html for reviewers.

## Output Object Schema (Recap)
```json
{
  "format": "markdown|html|json",
  "destination": "git:pr|git:issue|file:/path|log:stdout|log:stderr",
  "template": "table|text",          // omit or empty when format=json
  "fields": ["FieldA","FieldB"],      // optional subset of allowed fields
  "group_by": ["Field"],               // optional grouping
  "changes": ["new","changed"],       // optional change category filter (order: new, changed, removed, existing)
  "combined": true,                     // merge with other outputs (same dest+format)
  "collapse": true,                     // markdown/html collapsible section
  "title": "Custom Title",             // heading / section title
  "comment": "Context line under title"
}
```
For full field semantics see: [Policy System](../concepts/policy-system.md#output-configuration).

## Format Details
### Markdown
- Suitable for PR comments / issues.
- Templates:
  - `table`: Tabular layout (default if omitted)
  - `text`: Narrative sections; severity emojis for vulnerability/license findings
- Supports `collapse:true` (rendered as collapsible details block when combined or multi-section).

### HTML
- Mirrors markdown semantics but rendered with HTML templates.
- Useful when posting to systems that can render HTML (or exporting to static site / dashboard ingest pipeline).

### JSON
- Structured array of finding objects.
- `template` MUST be absent or empty. If provided non-empty → validation error.
- Key ordering not guaranteed; rely on keys, not order.
- Combined JSON (when multiple outputs share destination+format and set `combined:true`) yields a single concatenated array (stable policy iteration order). No wrapper object is introduced.

## Templates
| Template | Formats | Purpose |
|----------|---------|---------|
| table | markdown, html | Compact comparative view (columns = selected fields) |
| text | markdown, html | Narrative list style with icons / emphasis |
| (none) | json | Always raw structured output |

If `template` omitted for markdown/html → defaults to `table`.

## Field Selection (`fields`)
- Limits displayed columns (markdown/html) or object properties (json) for readability.
- If omitted: implementation default (may include a curated baseline per domain).
- Use only allowed domain fields (see Policy System field tables). Invalid field → validation error.

## Grouping (`group_by`)
- Groups findings hierarchically by specified fields.
- When `Severity` not included but present in group members, renderer may display highest severity in aggregate row.
- Empty or omitted = no grouping.

## Change Filtering (`changes`)
Restricts which change categories appear. Common PR pattern: `"changes":["new","changed"]` to avoid historical noise. See: [Diff-Based Analysis](../concepts/diff-analysis.md).

## Combined Outputs (`combined:true`)
Rules & edge cases centralised in: [Combining & Grouping](./combining-grouping.md). Key points:
- Homogeneous format per combined destination.
- JSON groups produce a single concatenated array.
- Mixed JSON with markdown/html not allowed.

## Collapse (`collapse:true`)
Applies to markdown/html tables or sections to reduce comment length. Ignored for JSON.

## Examples
Minimal markdown table:
```json
{"format":"markdown","template":"table","destination":"git:pr","fields":["VulnerabilityID","Severity"],"changes":["new"]}
```
Narrative text license report:
```json
{"format":"markdown","template":"text","destination":"git:pr","title":"License Issues","changes":["new","changed"],"fields":["Name","Category","Severity","PkgName"]}
```
JSON export (combined across multiple vulnerability policies):
```json
{"format":"json","destination":"file:/results/vuln-diff.json","combined":true,"changes":["new","changed","removed"]}
```

## AI Governance Rationale
Deterministic markdown & JSON outputs allow automated gating in fast, AI‑assisted development cycles: reviewers see only relevant new risk (filtered) while downstream tools ingest JSON to enforce dashboards / SLOs without parsing brittle human-formatted comments.

## Common Mistakes & Fixes
| Problem | Cause | Fix |
|---------|-------|-----|
| Template specified for JSON | `"template":"table"` with `"format":"json"` | Remove or set empty; JSON ignores templates |
| Mixed formats in combined group | `combined:true` outputs mix JSON with markdown/html | Use a pure JSON group or separate destinations |
| Empty PR comment | Only `existing` changes selected | Include `new` / `changed` in `changes` for PR outputs |
| Fields rejected | Invalid or domain-mismatched field | Use allowed fields table (Policy System reference) |
| Ungrouped clutter | Large ungrouped table | Add `group_by` (e.g., Severity, PkgName) |

## Best Practices
| Goal | Recommendation |
|------|---------------|
| Minimize review noise | Restrict PR outputs to `new` + `changed`; route `existing` to issue/file JSON |
| Justify blocking | Include explanatory fields (Severity, FixedVersion, Category) |
| Machine ingestion | Always produce at least one JSON output for critical policies |
| Readability | Use `text` template for concise narrative where tables become wide |
| Aggregation | Combine related policies for one PR comment using `combined:true` |

## Related Topics
- [Output Destinations](./destinations.md)
- [Combining & Grouping](./combining-grouping.md)
- [Policy System](../concepts/policy-system.md)
- [Diff-Based Analysis](../concepts/diff-analysis.md)
- [Configuration Overview](../configuration/overview.md)

---
Next: choose where these formats are delivered via [Output Destinations](./destinations.md).
