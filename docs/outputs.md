# Outputs

Each policy can have multiple outputs. Outputs control the format, destination, and presentation of results.

## Output Configuration

```json
{
  "format": "markdown",
  "destination": "git:pr",
  "template": "table",
  "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
  "changes": ["new", "changed"],
  "group_by": ["Severity"],
  "combined": false,
  "collapse": true,
  "title": "Security Issues",
  "comment": "Review required before merging"
}
```

---

## Formats

| Format | Description | Template Required |
|--------|-------------|-------------------|
| `markdown` | GitHub-flavored markdown tables or text | Yes: `table` or `text` |
| `html` | HTML formatted output | Yes: `table` or `text` |
| `json` | Structured JSON array | No (template must be empty) |

### Markdown/HTML Templates

- **`table`** (default): Compact table layout, one row per finding
- **`text`**: Narrative format with severity icons and structured sections

### JSON Output

JSON outputs are always arrays. Each policy produces an array of finding objects:

```json
[
  {
    "policy": "block-critical",
    "title": "Vulnerability Policy: block-critical",
    "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
    "sources": {
      "vulnerabilities": {
        "new": {
          "action": "block",
          "change": "new",
          "header": "Detection: new - Action: block",
          "items": [
            {
              "VulnerabilityID": "CVE-2024-0001",
              "PkgName": "lodash",
              "Severity": "CRITICAL",
              "FixedVersion": "4.17.21"
            }
          ]
        }
      }
    }
  }
]
```

**Important**: When using `format: json`, do not set `template`. It will cause a validation error.

---

## Destinations

| Destination | Description |
|-------------|-------------|
| `git:pr` | Post as PR comment (requires PR context) |
| `git:issue` | Create/update GitHub issue |
| `file:/path` | Write to file in `/results` volume |
| `log:stdout` | Print to standard output |
| `log:stderr` | Print to standard error |
| `url:https://...` | HTTP POST to webhook endpoint |

### git:pr

Posts or updates a comment on the pull request. Requires:
- `CI_EVENT=pr`
- `GITHUB_TOKEN`, `GITHUB_OWNER`, `GITHUB_REPO`, `GITHUB_PR_NR`

```json
{ "format": "markdown", "destination": "git:pr" }
```

### git:issue

Creates or updates a GitHub issue. Typically used for tracking existing backlog. Requires:
- `CI_EVENT=main`
- `GITHUB_TOKEN`, `GITHUB_OWNER`, `GITHUB_REPO`

```json
{ "format": "markdown", "destination": "git:issue", "changes": ["existing"] }
```

**Note**: Issue titles are suppressed (handled by GitHub).

### file:

Write to a file. Path is relative to `/results` mount:

```json
{ "format": "json", "destination": "file:/results/vulnerabilities.json" }
```

### log:stdout / log:stderr

Print to console (useful for debugging or simple CI logs):

```json
{ "format": "markdown", "destination": "log:stdout" }
```

### url:

HTTP POST to a webhook. Body is the rendered output. Content-Type is auto-detected (application/json for JSON outputs):

```json
{ "format": "json", "destination": "url:https://hooks.example.com/security" }
```

The `X-Report-Title` header is included when a title is available.

**Note**: Non-2xx responses are logged but don't affect exit code.

---

## Filtering by Change Category

Use `changes` to limit which findings appear:

```json
{ "changes": ["new", "changed"] }
```

| Category | When to Use |
|----------|-------------|
| `new` | PR comments — show introduced issues |
| `changed` | When attributes shifted (version, severity) |
| `removed` | Track remediation progress |
| `existing` | Backlog tracking (issues, JSON exports) |

**Common pattern**:
- PR comment: `["new", "changed"]` — actionable delta
- Issue: `["existing"]` — backlog tracking
- JSON export: all categories for full audit

---

## Field Selection

Limit which fields appear in output:

```json
{ "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"] }
```

If omitted, a default set of fields is included. See [Policies](./policies.md) for allowed fields per policy type.

---

## Grouping

Group findings by field values:

```json
{ "group_by": ["Severity"] }
```

Creates sections for each severity level. Multiple fields create nested grouping.

**Severity aggregation**: When grouping omits `Severity`, the highest severity in each group is shown in the header.

**Note**: Grouping doesn't affect JSON output (remains flat array). Process JSON client-side if needed.

---

## Combining Outputs

Multiple policies can write to the same destination. Use `combined: true` to merge them:

```json
{
  "vulnerability": [{
    "name": "critical",
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "combined": true
    }]
  }],
  "license": [{
    "name": "gpl",
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "combined": true
    }]
  }]
}
```

This produces a single PR comment with both vulnerability and license sections.

### Combining Rules

| Mix | Allowed |
|-----|---------|
| Markdown + Markdown | ✅ Merged into one document |
| HTML + HTML | ✅ Merged into one document |
| JSON + JSON | ✅ Single concatenated array |
| Markdown + HTML | ✅ Treated as separate groups |
| JSON + Markdown | ❌ **Error**: formats must match |

**JSON combining**: Produces a single array containing all policies' results. No wrapper object.

### Empty Combined Outputs

If all policies in a combined group produce zero results (after filtering), the entire output is suppressed.

---

## Collapsible Sections

For long reports, use `collapse: true`:

```json
{ "collapse": true }
```

Renders markdown/HTML sections as collapsible (using `<details>` tags). Ignored for JSON.

---

## Title and Comment

Customize the section header:

```json
{
  "title": "Critical Vulnerabilities",
  "comment": "These must be fixed before merging"
}
```

- `title`: Section heading
- `comment`: Explanatory text below the title

---

## Examples

### PR Comment with New Issues Only

```json
{
  "format": "markdown",
  "destination": "git:pr",
  "template": "table",
  "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
  "changes": ["new"],
  "collapse": true,
  "title": "New Vulnerabilities"
}
```

### JSON Export for Automation

```json
{
  "format": "json",
  "destination": "file:/results/security-report.json",
  "changes": ["new", "changed", "removed"],
  "combined": true
}
```

### Backlog Issue

```json
{
  "format": "markdown",
  "destination": "git:issue",
  "template": "table",
  "fields": ["VulnerabilityID", "PkgName", "Severity"],
  "changes": ["existing"],
  "title": "Security Backlog"
}
```

### Webhook Notification

```json
{
  "format": "json",
  "destination": "url:https://hooks.slack.com/services/...",
  "changes": ["new"],
  "title": "New Security Issues"
}
```

### Combined PR Comment (Multiple Policies)

```json
{
  "vulnerability": [{
    "name": "vulns",
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "combined": true,
      "title": "Vulnerabilities"
    }]
  }],
  "license": [{
    "name": "licenses", 
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "combined": true,
      "title": "License Issues"
    }]
  }]
}
```

---

## Common Mistakes

| Problem | Cause | Fix |
|---------|-------|-----|
| Empty PR comment | Only `existing` in changes | Include `new` or `changed` |
| Template error with JSON | Set `template` for JSON format | Remove `template` field |
| Mixed format error | `combined: true` with JSON + markdown | Use same format for all combined outputs |
| File not written | Path outside `/results` mount | Use absolute path like `/results/file.json` |
| No PR comment | Missing GitHub env vars | Set `GITHUB_TOKEN`, `GITHUB_PR_NR`, etc. |
| Invalid field error | Field not allowed for policy type | Check allowed fields in [Policies](./policies.md) |
