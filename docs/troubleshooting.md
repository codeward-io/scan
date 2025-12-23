# Troubleshooting

Solutions for common issues, plus FAQ, performance tips, and security notes.

## Quick Diagnosis

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| Everything marked as "new" | Missing baseline | Mount `/main`, set `CI_EVENT=pr` |
| No PR comment posted | Missing env vars | Set `GITHUB_TOKEN`, `GITHUB_PR_NR`, `GITHUB_OWNER`, `GITHUB_REPO` |
| Empty PR report | Wrong change filter | Include `new` or `changed` in `changes` |
| Mixed format combine error | JSON + markdown in combined group | Use same format for all combined outputs |
| Template error with JSON | `template` set for JSON output | Remove `template` field |
| Invalid field error | Field not available for policy type | Check allowed fields in [Policies](./policies.md) |
| Rules ignored | Wrong rules format | Use array: `"rules": [{ "field": "...", "type": "...", "value": "..." }]` |
| Relationship filter ignored | Dependency tree disabled | Set `global.dependency_tree: true` |
| Scan doesn't find packages | Dependencies not installed | Run `npm ci`, `pip install`, etc. before scanning |
| Validation rule ignored | Nested object instead of array | Use `"rules": [{ ... }]` array format |
| Permission denied on results | Container can't write | Ensure host directory is writable |

---

## Environment Variables

Complete reference for all environment variables:

### Required for Scanning

| Variable | Required | Description |
|----------|----------|-------------|
| `CODEWARD_CI` | Recommended | Set to `true` to enable CI behaviors |
| `CODEWARD_CI_EVENT` | Yes | `pr` for diff mode, `main` for single-branch scan |

### GitHub Integration

| Variable | Required For | Description |
|----------|--------------|-------------|
| `CODEWARD_GITHUB_TOKEN` | `git:pr`, `git:issue` | GitHub API token |
| `CODEWARD_GITHUB_OWNER` | `git:pr`, `git:issue` | Repository owner/organization |
| `CODEWARD_GITHUB_REPO` | `git:pr`, `git:issue` | Repository name |
| `CODEWARD_GITHUB_PR_NUMBER` | `git:pr` | Pull request number |

### Configuration

| Variable | Required | Description |
|----------|----------|-------------|
| `CODEWARD_CONFIG_PATH` | No | Override config file path |
| `CODEWARD_PRIVATE_CONFIG_PATH` | No | Path to private config overrides |

### Docker Volumes

| Mount | Purpose |
|-------|---------|
| `/main` | Base branch (main) checkout — required |
| `/branch` | Feature branch checkout — required for PR mode |
| `/results` | Output files (reports, JSON) |
| `/.cache` | Trivy database and scan cache |

---

## Common Issues

### Everything Shows as "New"

The scanner can't find the baseline to compare against.

**Fix:**
1. Mount the main branch at `/main`
2. Mount the feature branch at `/branch`
3. Set `CI_EVENT=pr`

```bash
docker run --rm \
  -v /path/to/main:/main:rw \
  -v /path/to/branch:/branch:rw \
  -e CI_EVENT=pr \
  ghcr.io/codeward-io/scan:latest
```

### No PR Comment Appears

The scanner can't post to GitHub.

**Check:**
1. `GITHUB_TOKEN` is set and has `pull-requests: write` scope
2. `GITHUB_PR_NR` is set to the PR number
3. `GITHUB_OWNER` and `GITHUB_REPO` are correct
4. `CI_EVENT=pr`

### Empty Output

No findings matched your policy after filtering.

**Check:**
1. `changes` array includes `new` (not just `existing`)
2. Rules aren't too restrictive
3. The diff actually introduces findings (try with fewer rules)

### Validation Errors

Common config validation issues:

| Error | Fix |
|-------|-----|
| `invalid format` | Use `markdown`, `html`, or `json` |
| `invalid destination` | Start with `git:`, `log:`, `file:`, or `url:` |
| `template must be empty` | Remove `template` for JSON outputs |
| `invalid field` | Use only allowed fields for the policy type |
| `missing required field` | Add `name`, `actions`, `rules` to policy |

---

## Performance Tips

### Caching

Mount a persistent cache directory for faster subsequent scans:

```bash
-v /persistent/cache:/.cache:rw
```

The cache stores:
- Trivy vulnerability database
- Scan metadata for faster reprocessing

### Dependency Installation

For accurate transitive dependency detection, install dependencies before scanning:

```bash
# In your CI before running Codeward
npm ci
# or
pip install -r requirements.txt
# or
go mod download
```

### Reducing Scan Time

1. **Start with focused policies**: Scan only CRITICAL first, expand later
2. **Disable dependency tree**: Only enable if using `Relationship`, `Parents`, `Children` filters
3. **Limit outputs**: Reduce number of output configurations

---

## Security Notes

### Data Handling

- All scanning runs inside the container — no code leaves the runner
- Outputs go only to configured destinations
- No external network calls except:
  - Trivy database updates
  - Your configured `url:` destinations

### Token Security

- Use repository-scoped tokens, not personal access tokens
- Grant minimal required permissions:
  - `contents: read` — checkout
  - `packages: read` — pull scanner image
  - `pull-requests: write` — post comments (if using `git:pr`)
  - `issues: write` — create issues (if using `git:issue`)

### Cache Contents

- Trivy database (vulnerability data)
- Scan metadata
- **No credentials** are cached

---

## FAQ

### Why only diff findings by default?

Focusing on newly introduced issues means:
- Developers see only what they need to fix
- Existing backlog doesn't block PRs
- Remediation is tracked separately

### Can I scan the full codebase?

Yes. Use `CI_EVENT=main` for single-branch mode, and include `existing` in your `changes` filter.

### How do I allow-list a specific CVE?

Use an inequality rule:

```json
{
  "rules": [
    { "field": "Severity", "type": "eq", "value": "CRITICAL" },
    { "field": "VulnerabilityID", "type": "ne", "value": "CVE-2024-ACCEPTED" }
  ]
}
```

### How do I block only direct dependencies?

Enable dependency tree and filter by relationship:

```json
{
  "global": { "dependency_tree": true },
  "vulnerability": [{
    "operator": "AND",
    "rules": [
      { "field": "Severity", "type": "eq", "value": "CRITICAL" },
      { "field": "Relationship", "type": "eq", "value": "direct" }
    ]
  }]
}
```

### Is the JSON schema stable?

Yes. JSON outputs are always arrays with stable structure. New fields may be added (additive changes only).

### How do I post to Slack/Teams/other webhook?

Use the `url:` destination:

```json
{ "format": "json", "destination": "url:https://hooks.slack.com/..." }
```

Process the payload with your webhook handler.

### Why no SARIF export?

Codeward focuses on actionable, diff-aware output. SARIF can introduce noise. JSON outputs provide equivalent data for automation.

---

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success — no blocking findings |
| 1 | Failure — at least one `block` action triggered, or fatal error |

---

## Getting Help

If you're stuck:

1. Check the error message — Codeward provides specific validation errors
2. Review this troubleshooting guide
3. Check your config against [Configuration](./configuration.md)
4. Verify allowed fields in [Policies](./policies.md)

---

## Future Roadmap

Planned features (not yet implemented):

- **Codeward.io API integration**: Central dashboard for vulnerability tracking
- **AI enrichment**: AI-powered context and recommendations for findings

These features will be documented when available.
