# Glossary

Canonical definitions to reduce ambiguity.

## Actions
`info | warn | block` – Policy result severities. `block` sets non-zero exit code if any blocking findings remain.

## Change Categories
Order: `new, changed, removed, existing`.
`new` – Introduced on this branch.
`changed` – Present in base; attributes differ (e.g., version, severity-relevant fields).
`removed` – Present in base but deleted on this branch.
`existing` – Unchanged (usually excluded from PR outputs; may appear in JSON inventory).

## Policy
Configuration object with `name`, `actions`, optional `rules`, and `outputs` governing one intent.

## Rule
Filter predicate object (`field`,`type`,`value`) (non-validation) or assertion (`type`,`key/value/path`) (validation) evaluated OR-wise within a policy.

## Diff-Aware
Only evaluates net-new or changed findings vs a baseline (default branch) to suppress backlog noise.

## Destination
Configured output target: file, stdout/stderr, git:pr, git:issue, url (POST), etc.

## Combined Output
Multiple policy outputs with `combined:true` targeting the same destination merged into a single artifact. For JSON this is one concatenated array (no wrapper). See [Combining & Grouping](../output/combining-grouping.md).

## Grouping
Aggregation of findings (e.g., by field set) or merging of multiple policies into a single rendered section.

## Validation Policy
Policy evaluating filesystem or structured file content (text/json/yaml/filesystem) using assertion rule types (e.g., `exists`, `regex`).

## Vulnerability / License / Package Policies
Domain policies operating on scanner-provided records (CVE, license metadata, dependency entries).

## Progressive Enforcement
Staged rollout: begin with informational outputs, escalate to warnings, then enable blocking only for patterns you are confident about.

## Uniform JSON Array
All JSON outputs are arrays of objects with stable ordering. Combined JSON arrays append objects in deterministic order.

## Cache
Local artifacts (scanner DB, dependency metadata) enabling faster warm scans and reduced network pulls.

See also: [Diff Analysis](diff-analysis.md), [Policy System](policy-system.md), [Progressive Enforcement](../operations/progressive-enforcement.md), [Combining & Grouping](../output/combining-grouping.md).
