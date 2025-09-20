# Architecture

## Stages
### 1. Scans
Invokes underlying scanners / collectors (e.g., dependency metadata). Raw results separated per domain.

### 2. Result Sets
Two perspectives: base (e.g., default branch) and head (PR branch). Stored transiently.

### 3. Diff Engine
Classifies findings into `new, changed, removed, existing` (canonical order). Typically only `new` and `changed` are surfaced in PR outputs.

### 4. Policy Evaluation
Each policy rule runs filter logic over diff findings. Matching findings labeled with the rule action (`info | warn | block`).

### 5. Grouping & Combination
Optionally merges multiple policy outputs into consolidated tables / JSON arrays.

### 6. Rendering
Markdown / JSON / other format serialization with deterministic ordering.

### 7. Destinations
Writes to comment, files, stdout, or remote webhook via `url` destination.

### 8. Exit Code
If any rule yields `block`, process returns non-zero (usually failing CI job).

## Key Properties
- Diff-aware: avoids legacy backlog noise
- Deterministic output ordering
- Combined JSON: single concatenated array per destination (uniform format)
- Extensible: new policy domains can plug into Policy Evaluation stage

See also: [Policy System](policy-system.md), [Output Formats](../output/formats.md), [Security & Trust Model](../operations/security-trust-model.md), [Combining & Grouping](../output/combining-grouping.md).
