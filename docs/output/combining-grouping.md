---
title: Combining & Grouping
description: Canonical rules for merging policy outputs and grouping findings across diff-aware reports.
---
# Combining & Grouping

Central reference for merging outputs and grouping findings. Canonical change category order: `new, changed, removed, existing` (see [Style & Naming Guide](../configuration/style-naming-guide.md)).

## Combining Outputs
Multiple policy outputs can target one destination. A combined set is defined by identical (destination, format) with `combined:true` on each member.

| Allowed Mix | Result |
|-------------|--------|
| All markdown | Single merged markdown section (per-policy subsections / tables in order) |
| All html | Single merged HTML section |
| All JSON | Single concatenated JSON array (records appended in policy order) |
| Markdown + HTML | Treated as separate combined groups (format boundary) |
| JSON + Markdown/HTML | Not allowed (format homogeneity required) |

### JSON Semantics
All JSON outputs are arrays of objects. Combined JSON appends elements sequentially into a single concatenated array (no wrapper object). Field order inside objects not guaranteed (treat as unordered key set). Stable ordering of records follows policy evaluation order then record natural ordering.

### Empty Combined Artifacts
If every component output for a (destination, format) group yields zero rows (after `changes` filtering) the renderer may suppress the entire combined artifact (avoids empty PR comments). At least one non-empty member → header and sections render.

## Grouping Findings (`group_by`)
`group_by` lists fields that define hierarchical grouping for markdown / HTML table or text templates. Behavior:
- Each grouping field introduces a nested section level.
- Severity aggregation: when `Severity` omitted from subgroup rows, renderer may surface highest severity at group header (implementation detail; do not rely for automation).
- Grouping does not alter JSON output (JSON remains flat array). To emulate grouping programmatically, client code can post-process by keying on the same field list.

### Grouping Example
```json
{"format":"markdown","template":"table","destination":"git:pr","fields":["Severity","PkgName","VulnerabilityID"],"group_by":["Severity"],"changes":["new","changed"]}
```
Produces severity‑segmented tables for new / changed vulnerabilities.

## Best Practices
| Goal | Recommendation |
|------|---------------|
| Reduce PR noise | Combine related policies into one PR comment (one markdown group) |
| Automation clarity | Keep JSON outputs separate if downstream systems need domain partitioning; otherwise combine for single ingest step |
| Avoid width overflow | Use grouping instead of adding >6 columns to a table |
| Deterministic ingestion | Prefer one combined JSON export per domain with stable field set |
| Readability | Use `collapse:true` on secondary sections inside a large combined comment |

## Common Errors
| Message | Cause | Fix |
|---------|-------|-----|
| Mixed format in combined destination | Attempted JSON + markdown mix | Split into two destinations |
| Empty combined output | All component outputs filtered to zero rows | Confirm diff introduces findings or remove group |
| Fields rejected | Unsupported field name in `fields` or `group_by` | Use allowed field list (Policy System) |

## Related References
- [Output Formats](formats.md)
- [Destinations](destinations.md)
- [Policy System](../concepts/policy-system.md)
- [Glossary](../concepts/glossary.md)

---
Use combining to keep reviewer focus tight while still generating comprehensive machine-readable JSON arrays.
