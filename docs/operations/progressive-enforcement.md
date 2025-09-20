---
title: Progressive Enforcement
description: Phased approach for safely rolling out diff-aware policy gating from observation to blocking net-new critical risk.
---

# Progressive Enforcement

Adopt policy gating in safe phases to build confidence before blocking. Change categories referenced (`new`, `changed`, `removed`, `existing`) follow the canonical semantics in [Diff-Based Analysis](../concepts/diff-analysis.md).

## Phase Table
| Phase | Rule Actions | Purpose |
|-------|--------------|---------|
| 1 | info | Observe & tune noise |
| 2 | warn | Signal risk; non-blocking |
| 3 | block (critical), warn (others) | Prevent high-impact regressions |
| 4 | block (critical+high) | Tighten gate as backlog shrinks |
| 5 | block (severity thresholds per domain) | Mature steady-state |

## Recommendations
- Start with a single high-signal vulnerability rule (e.g., CRITICAL `new` only)
- Add license / package policies once initial noise is characterized
- Promote to `block` only after two+ weeks of stable signal

## Rollback Guidance
If false positives spike, demote affected rule to `warn` while triaging root cause.

## Communication
Document enforcement phase in PR comment or README so developers know expectations.

See also: [Glossary](../concepts/glossary.md), [Policy System](../concepts/policy-system.md), [Combining & Grouping](../output/combining-grouping.md).
