# Troubleshooting & FAQ

Centralized solutions for common setup and policy issues.

## Quick Diagnosis Table
| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| Everything marked new | Missing baseline mount or `CI_EVENT` not `pr` | Mount baseline (e.g. main) and set `CI_EVENT=pr` |
| No PR comment posted | Missing `GITHUB_PR_NR` / insufficient token perms | Provide required env vars; ensure `GITHUB_TOKEN` has comment scope |
| Mixed format combine error | JSON + markdown in same combined group | See Combining & Grouping; split destination or unify format |
| Empty PR report | Only `existing` selected in `changes` | Include `new` / `changed` change categories |
| Relationship filter ignored | `dependency_tree` disabled | Enable `global.dependency_tree=true` in config |
| Template error with JSON | `template` specified for JSON output | Remove `template` key for JSON |
| Unsupported field error | Field not in allowed domain list | Use canonical field list in Policy System |
| Validation rule ignored | Nested object instead of array | Use `"rules": [ { ... }, { ... } ]` |
| Combined JSON missing data | One policy produced no diff findings | Confirm diff introduces net-new or remove empty policy from group |

## Baseline / Diff Issues
Ensure two mounts (baseline + head). In GitHub Actions the action handles this automatically; custom Docker runs must supply both.

## CI Environment Variables
Minimal for PR diff posting:
- `CI_EVENT=pr`
- `GITHUB_TOKEN`
- `GITHUB_PR_NR`
- `GITHUB_OWNER`, `GITHUB_REPO`

Without these git destinations are skipped; other outputs still render.

## Performance Questions
First run (cold) slower; subsequent warm scans benefit from caching. See Performance & Caching.

## JSON Schema Stability
JSON outputs are uniform arrays. Schema additions are backward compatible; removals only in major version releases.

## FAQ
### Why only diff findings by default?
Focus developers on newly introduced risk and accelerate remediation without backlog noise.
### Can I scan full history or entire codebase?
Yes: run a separate branch scan with full mode or toggle an environment flag if supported.
### How do I allowlist one CVE?
Add a `ne` rule on VulnerabilityID or narrow severity rules; document rationale in output comment.
### How do I suppress a single finding?
Adjust or refine policy filters (e.g., restrict severity or package name). Granular suppression lists currently not provided (roadmap consideration).
### Why no SARIF export?
Prioritizing concise markdown + uniform JSON arrays. SARIF may introduce noise; may be revisited if demand is strong.
### Is the JSON schema stable?
Yes—additive changes only (fields appended). Breaking field removals avoided.
### How do I post results to an external system?
Use a `url` destination. All outputs in the combined group must be JSON if any are JSON.
### How to block only direct vulnerabilities?
Enable `dependency_tree` globally and filter `Relationship=direct`.
### How to suppress existing license debt?
Route `existing` to an issue or JSON file; exclude from PR outputs via `changes`.

See also: [Combining & Grouping](../output/combining-grouping.md), [Performance & Caching](performance-caching.md), [Security & Trust Model](security-trust-model.md), [Glossary](../concepts/glossary.md).
