# Docker Installation

Run **Codeward** via Docker for a consistent, reproducible scanning environment across any CI system (GitHub Actions, GitLab, Jenkins, local dev). Use Docker when you need tighter control over dependency installs, caching, or custom orchestration.

> Governance & AI Context: Containerized execution inserts a lightweight, deterministic gate so large or AI‑generated commits are checked for new vulnerable dependencies, license drift, and structural issues before merge.

## Quick Scan (Single Path)
Scans a single repository state (main branch / default branch context). Suitable for scheduled baseline scans.
```bash
docker run --rm \
  -v /abs/path/to/repo:/main:rw \
  -v /abs/path/to/results:/results:rw \
  -v /abs/path/to/cache:/.cache:rw \
  -e CI=true -e CI_EVENT=main \
  ghcr.io/codeward-io/scan:v0.0.1
```
Pin an explicit version for reproducibility (`:v0.0.1`). `:latest` is fine for short experiments.

## Diff Scan (PR / Change Review)
Compare a feature branch checkout to the main branch to focus only on `new`, `changed`, `removed`, `existing` items (see [Diff-Based Analysis](../concepts/diff-analysis.md)).
```bash
docker run --rm \
  -v /abs/path/to/main-checkout:/main:rw \
  -v /abs/path/to/feature-checkout:/branch:rw \
  -v /abs/path/to/results:/results:rw \
  -v /abs/path/to/cache:/.cache:rw \
  -e CI=true -e CI_EVENT=pr \
  -e GITHUB_TOKEN=$GITHUB_TOKEN \
  -e GITHUB_OWNER=myorg -e GITHUB_REPO=myrepo -e GITHUB_PR_NR=123 \
  ghcr.io/codeward-io/scan:v0.0.1
```
`GITHUB_*` variables are only required when your configuration includes `git:pr` or `git:issue` destinations.

## Recommended Volume Layout
| Host Path | Container Mount | Purpose |
|-----------|-----------------|---------|
| /abs/.../main | /main | Baseline repository (main branch) |
| /abs/.../feature | /branch | Feature branch (diff scans only) |
| /abs/.../results | /results | Report artifacts (if configured) |
| /abs/.../cache | /.cache | Trivy & internal cache (improves speed) |

## Optional Configuration Overrides
```bash
docker run --rm \
  -v $(pwd)/config.json:/config/config.json:ro \
  -v $(pwd)/private.json:/config/private.json:ro \
  -v $(pwd)/repo-main:/main \
  -v $(pwd)/repo-feature:/branch \
  -v $(pwd)/results:/results \
  -v $(pwd)/cache:/.cache \
  -e CONFIG_PATH=/config/config.json \
  -e PRIVATE_CONFIG_PATH=/config/private.json \
  -e CI=true -e CI_EVENT=pr \
  ghcr.io/codeward-io/scan:v0.0.1
```
If omitted, embedded defaults (and any `config.json` discovered in repo paths) are used.

## Key Environment Variables
| Variable | Required | Description |
|----------|----------|-------------|
| CI | No (recommended) | Enables CI behaviors; set to `true` in pipelines |
| CI_EVENT | Yes | `pr` for diff mode, `main` for single path scan |
| GITHUB_TOKEN | For git destinations | Auth for PR comment / issue creation |
| GITHUB_OWNER | For git destinations | Repository owner/org |
| GITHUB_REPO | For git destinations | Repository name |
| GITHUB_PR_NR | PR mode | Pull request number (when CI_EVENT=pr) |
| CONFIG_PATH | Optional | Path to primary config file |
| PRIVATE_CONFIG_PATH | Optional | Path to private config file |

Dependency graph filtering (e.g., `Relationship`, Parents / Children fields) requires enabling `global.dependency_tree=true` in configuration (see [Configuration Overview](../configuration/overview.md)).

## Performance Tips
- Reuse a persistent cache volume for faster vulnerability DB updates.
- Install dependencies (e.g., `npm ci`, `pip install -r requirements.txt`) in the host checkout before running the container for richer transitive results.
- Keep config outputs focused (limit fields / disable unused policies) to reduce render time.
- Pin image versions to avoid unplanned drift; upgrade intentionally.

## Minimal Example Policy Snippet
```json
{
  "vulnerability": [
    {
      "name": "critical-high",
      "actions": {"new": "block", "existing": "warn"},
      "rules": [
        {"field": "Severity", "type": "eq", "value": "CRITICAL"},
        {"field": "Severity", "type": "eq", "value": "HIGH"}
      ],
      "outputs": [
        {"format": "markdown", "template": "table", "destination": "file:/results/vulns.md", "fields": ["VulnerabilityID","PkgName","Severity","FixedVersion"], "changes": ["new"], "collapse": true}
      ]
    }
  ]
}
```
(For clearer progressive enforcement prefer separate policies: one blocking CRITICAL, one observing HIGH.)

## Troubleshooting
Most runtime / classification issues are centralized in the [Troubleshooting & FAQ](../operations/troubleshooting-faq.md). Only unique container-specific notes are listed here:
| Symptom | Cause | Resolution |
|---------|-------|------------|
| Permission denied writing results | Host directory not writable by container user | Adjust ownership or omit a custom `--user` flag |
| Cache not reused | Ephemeral cache path | Mount persistent host dir to `/.cache` |
| Everything marked new | Missing `/main` mount or wrong `CI_EVENT` | Mount baseline at `/main` and set `CI_EVENT=pr` |

See FAQ for: empty outputs, mixed-format combine errors, PR comment absence, etc.

## Best Practices
| Goal | Recommendation |
|------|---------------|
| Reproducibility | Pin the image tag instead of using `:latest` |
| Low-noise PRs | Restrict PR markdown outputs to `new` (and essential `changed`) |
| Automation | Produce one combined JSON artifact (concatenated array) — see [Combining & Grouping](../output/combining-grouping.md) |
| Dependency graph filters | Enable `global.dependency_tree` before using relationship fields |
| Token hygiene | Use repo-scoped, least-privilege token for git destinations |

## Security Notes
- Treat mounted config files as sensitive (policy logic can reveal internal criteria).
- Use least-privilege GitHub tokens (avoid personal tokens where possible).
- Avoid piping untrusted dynamic JSON into configs without validation.

## Related Topics
- [GitHub Actions](./github-actions.md)
- [Configuration Overview](../configuration/overview.md)
- [Combining & Grouping](../output/combining-grouping.md)
- [Starter Configs](../examples/starter-configs.md)

---
Next: define or extend policies — see the [Policy System](../concepts/policy-system.md).
