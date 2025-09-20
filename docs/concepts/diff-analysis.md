# Diff-Based Analysis

Codeward focuses every policy and output on what a change introduces or alters— not historic backlog—when producing PR-centric views. Baseline or scheduled scans still enumerate the full existing backlog so you can track and remediate legacy issues separately without polluting review noise. A pull request (or feature branch) is compared to *main* and each record is classified so governance targets only net-new or modified risk.

> See also: [Glossary](./glossary.md) for term definitions and the [Policy System](./policy-system.md) for how actions are applied.

## How It Works
1. Scan baseline repository (mounted at `/main`).
2. Scan change workspace (mounted at `/branch`) when `CI_EVENT=pr`; otherwise single-path (main) scan.
3. Classify each record into one diff category (canonical order below).
4. Apply policy `actions` per category (only keys you define matter).
5. Render each output restricted to its `changes` filter.

## Change Categories (Authoritative)
| Category | Definition | Typical Action Pattern | Governance Rationale |
|----------|------------|------------------------|----------------------|
| new | Present only in branch | [block](./glossary.md#actions) / [warn](./glossary.md#actions) | Net-new risk entering codebase (highest leverage gate) |
| changed | Present in both but key attributes differ (e.g., version) | [warn](./glossary.md#actions) | Possible upgrade/downgrade or license/severity shift requiring review |
| removed | Present only in main (eliminated by branch) | [info](./glossary.md#actions) | Indicates remediation / improvement; never block |
| existing | Present in both unchanged | [info](./glossary.md#actions) / [warn](./glossary.md#actions) | Backlog context; avoid blocking velocity until ready to escalate |

(Glossary entries: [new](./glossary.md#change-categories) · [changed](./glossary.md#change-categories) · [removed](./glossary.md#change-categories) · [existing](./glossary.md#change-categories))

(Always express keys in the canonical order: `new`, `changed`, `removed`, `existing` for tables, examples, and action maps when ordering is shown.)

## Output Scoping with `changes`
Use the per-output `"changes": [ ... ]` array to minimize reviewer noise:
- PR comment: `["new","changed"]` (focus only on what to act on now).
- Backlog / issue: `["existing"]` (track inherited debt separately).
- Progress reporting: `["removed"]` (celebrate remediation).
- Full audit artifact: all four categories.

## Example Policy (Severity Gate)
```json
{
  "vulnerability": [
    {
      "name": "critical-and-high-vulns",
      "actions": {"new": "block", "changed": "warn", "removed": "info", "existing": "warn"},
      "rules": [
        {"field": "Severity", "type": "eq", "value": "CRITICAL"},
        {"field": "Severity", "type": "eq", "value": "HIGH"}
      ],
      "outputs": [
        {"format": "markdown", "template": "table", "destination": "git:pr", "fields": ["VulnerabilityID","PkgName","Severity","FixedVersion"], "changes": ["new","changed"], "collapse": true},
        {"format": "json", "destination": "file:/results/vuln-changes.json", "changes": ["new","changed","removed" ]}
      ]
    }
  ]
}
```

## Best Practices
| Goal | Practice |
|------|----------|
| Reduce PR noise | Exclude `existing` from PR destinations |
| Encourage remediation | Separate `removed` section or output |
| Highlight risky deltas | Always include `changed` for packages / licenses where version or classification can shift risk |
| Deterministic automation | Emit JSON with explicit `changes` filters; combine via [Combining & Grouping](../output/combining-grouping.md) |
| Safe rollout | Start blocking only on `new` critical risk, later expand |

## Common Misconfigurations
| Symptom | Cause | Fix |
|---------|-------|-----|
| Everything marked new | Baseline not mounted at `/main` or missing `CI_EVENT=pr` | Mount both `/main` & `/branch`; set `CI_EVENT=pr` |
| No changed items appear | Actual record attributes identical | Expected; verify version or metadata truly changed |
| Existing overwhelms review | PR output omitted `changes` filter | Add `"changes": ["new","changed"]` to PR outputs |

## Related Topics
- [Policy System](./policy-system.md)
- [Scanning Types](./scanning-types.md)
- [Combining & Grouping](../output/combining-grouping.md)
- [Output Formats](../output/formats.md)

---
Next: define rules & actions in the [Policy System](./policy-system.md).
