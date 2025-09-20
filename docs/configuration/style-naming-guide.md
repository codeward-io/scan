---
id: style-naming-guide
title: Style & Naming Guide
description: Canonical style rules: change category order, actions formatting, capitalization, severity ordering, combined JSON semantics.
keywords:
  - style
  - naming
  - conventions
  - documentation
  - governance
---
<!-- filepath: /Users/tambet/Documents/GitHub/codeward-io/docs/docs/configuration/style-naming-guide.md -->
# Style & Naming Guide

Consistency rules for documentation and configuration examples.

## Change Categories Order
Canonical order (always list in this sequence): `new, changed, removed, existing`.

## Actions Formatting
Always show as: `info | warn | block`.

## Capitalization
- Product: Codeward (capitalize first letter only)
- Domains: vulnerability, license, package, validation (lowercase mid-sentence)
- Use Oxford comma in lists.

## JSON Examples
- Use array form for `rules`.
- Omit `template` in JSON outputs (only markdown/html need it).
- Prefer one policy per intent; multiple small policies over one large.

## Combined JSON Semantics
All JSON outputs are arrays. When `combined:true`, outputs targeting the same JSON destination merge into a single concatenated array (no wrapper object, no per‑policy nesting). Mixed formats in a combined group are invalid—see [Combining & Grouping](../output/combining-grouping.md).

## Severity Ordering
When listed: critical, high, medium, low (descending risk). Use lowercase.

## Wording Preferences
- Use diff‑aware (hyphen) not diff based.
- Prefer "policy" over "rule set"; a policy contains rules.
- Use "combined JSON" not "merged object".
- Use "progressive enforcement" for staged rollout; avoid synonyms (gradual gating, phased blocking).

## Cross-References
- Definitions: [Glossary](../concepts/glossary.md)
- Diff semantics: [Diff-Based Analysis](../concepts/diff-analysis.md)
- Policy mechanics: [Policy System](../concepts/policy-system.md)
- Output combining: [Combining & Grouping](../output/combining-grouping.md)

---
Adhering to this guide keeps examples deterministic and reduces reviewer cognitive load.
