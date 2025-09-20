# Output Destinations

Where rendered outputs go. For formatting & templates see: [Output Formats](./formats.md). Combining semantics: [Combining & Grouping](./combining-grouping.md).

## Supported Destinations
| Destination Prefix | Example | Purpose |
|--------------------|---------|---------|
| git:pr | git:pr | Post / update pull request comment (requires PR context) |
| git:issue | git:issue | Create / update issue (repository backlog tracking) |
| file: | file:/results/report.md | Persist artifact in mounted `/results` volume |
| log:stdout | log:stdout | Print to standard output (CI logs) |
| log:stderr | log:stderr | Print to error stream |

Destination string format: `<prefix>[:<path>]` where `git:pr` / `git:issue` ignore path; `file:` requires absolute or relative path inside writable mount; `log:` variants ignore path.

## Examples
Minimal PR table:
```json
{"format":"markdown","template":"table","destination":"git:pr","fields":["VulnerabilityID","Severity"],"changes":["new"]}
```
Issue (backlog) plus PR (delta) split:
```json
[
  {"format":"markdown","template":"table","destination":"git:pr","changes":["new","changed"],"title":"New / Changed High+ Vulns"},
  {"format":"markdown","template":"table","destination":"git:issue","changes":["existing"],"title":"Existing High+ Vulns Backlog","collapse":true}
]
```
File JSON export:
```json
{"format":"json","destination":"file:/results/vuln-diff.json","changes":["new","changed","removed"],"combined":true}
```
Console (stdout) license summary:
```json
{"format":"markdown","template":"text","destination":"log:stdout","changes":["new"],"title":"License Changes"}
```

## Combined Destinations
`combined:true` aggregates multiple policy outputs targeting the same (destination, format) pair into a single artifact. Homogeneous format required (JSON groups yield one concatenated array; markdown groups produce one merged comment section). Full rules: [Combining & Grouping](./combining-grouping.md).

Example combining vulnerability + license into one PR comment:
```json
{
  "vulnerability": [
    {"name":"crit-block","actions":{"new":"block"},"rules":[{"field":"Severity","type":"eq","value":"CRITICAL"}],
     "outputs":[{"format":"markdown","template":"table","destination":"git:pr","combined":true,"title":"Critical Vulnerabilities"}]}],
  "license": [
    {"name":"gpl-block","actions":{"new":"block"},"rules":[{"field":"Name","type":"contains","value":"GPL"}],
     "outputs":[{"format":"markdown","template":"table","destination":"git:pr","combined":true,"title":"GPL License Issues"}]}]
}
```

## Environment Requirements (Git Destinations)
| Variable | Required For | Notes |
|----------|--------------|-------|
| GITHUB_TOKEN | git:pr, git:issue | API token with comment / issue scope |
| GITHUB_OWNER | git:pr, git:issue | Repository owner |
| GITHUB_REPO | git:pr, git:issue | Repository name |
| GITHUB_PR_NR | git:pr | PR number (when `CI_EVENT=pr`) |
| CI_EVENT | git:pr | Must be `pr` for diff; `main` skips PR posting |

Without required env vars a destination is skipped (others still process).

## Change Category Strategy
Typical mapping:
- PR (`git:pr`): `new`, `changed` — actionable delta
- Issue (`git:issue`): `existing` — backlog visibility
- File / JSON: include `removed` for audit/history

See: [Diff-Based Analysis](../concepts/diff-analysis.md).

## AI Governance Rationale
Destination segregation enables governance without overwhelming reviewers: PR comment shows only net‑new critical risk while backlog issues track legacy debt; JSON files power automated dashboards / compliance checks for AI‑accelerated change velocity.

## Common Mistakes & Fixes
| Problem | Cause | Fix |
|---------|-------|-----|
| Empty PR comment | Only `existing` changes selected | Include `new` / `changed` in `changes` for PR outputs |
| Mixed formats in combined group | Combined group mixes JSON with markdown/html | Use homogeneous format groups |
| Destination validation error | Invalid prefix (e.g. `stdout:`) | Use `log:stdout` or `log:stderr` |
| File not written | Path not in mounted writable volume | Ensure `file:` path resolves within `/results` (or write mount) |
| PR not posted | Missing required GitHub env vars | Provide `GITHUB_PR_NR`, `CI_EVENT=pr`, token, owner, repo |

## Best Practices
| Goal | Recommendation |
|------|---------------|
| Minimize PR noise | Limit PR outputs to delta (`new` + `changed`) |
| Persistent backlog | Route `existing` issues to a single `git:issue` combined report |
| Automation / ingestion | Generate at least one JSON file per critical domain |
| Traceability | Include JSON artifact path in CI summary |
| Clarity | Use `title` + concise `comment` to contextualize each combined section |

## Related Topics
- [Output Formats](./formats.md)
- [Combining & Grouping](./combining-grouping.md)
- [Policy System](../concepts/policy-system.md)
- [Configuration Overview](../configuration/overview.md)
- [Diff-Based Analysis](../concepts/diff-analysis.md)

---
Next: refine which fields & rules feed these outputs via the [Policy System](../concepts/policy-system.md).
