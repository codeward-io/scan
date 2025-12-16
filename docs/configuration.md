# Configuration

Codeward is configured with a single JSON file (`.codeward.json`) in your repository root. This file defines what to scan, what rules to apply, and where to send results.

## Config File Location

Codeward looks for configuration in this order:
1. Path specified by `CONFIG_PATH` environment variable
2. `.codeward.json` in repository root
3. Built-in defaults (if no config found)

## Top-Level Structure

```json
{
  "global": {
    "dependency_tree": false
  },
  "vulnerability": [ /* vulnerability policies */ ],
  "license": [ /* license policies */ ],
  "package": [ /* package policies */ ],
  "validation": [ /* validation policies */ ],
  "pr_validation": [ /* PR validation policies */ ]
}
```

All policy arrays are optional. Omit any you don't need.

### Global Settings

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `dependency_tree` | boolean | `false` | Enable dependency graph analysis. Adds `Relationship`, `Parents`, `Children`, `Targets` fields. Slight performance cost. |

## Policy Structure

Every policy (except validation types) follows this structure:

```json
{
  "name": "policy-name",
  "disabled": false,
  "actions": {
    "new": "block",
    "existing": "warn",
    "removed": "info",
    "changed": "warn"
  },
  "operator": "OR",
  "rules": [
    { "field": "Severity", "type": "eq", "value": "CRITICAL" }
  ],
  "outputs": [
    { /* output config */ }
  ]
}
```

### Policy Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier for the policy |
| `disabled` | No | Set `true` to skip this policy |
| `actions` | Yes | What to do for each change category |
| `operator` | No | `OR` (default) or `AND` for rule matching |
| `rules` | Yes | Array of filter rules (can be empty for "match all") |
| `outputs` | Yes | Where and how to report results |

## Change Categories

Codeward classifies every finding based on the diff between branches:

| Category | Meaning | Typical Action |
|----------|---------|----------------|
| `new` | Only in feature branch (introduced) | `block` or `warn` |
| `changed` | In both, but attributes differ | `warn` |
| `removed` | Only in main branch (deleted) | `info` |
| `existing` | In both, unchanged | `info` or `warn` |

**Key insight**: Only findings in `new` category should typically block PRs. Existing issues are technical debt to address separately.

## Actions

| Action | Effect |
|--------|--------|
| `info` | Log only, no impact on CI |
| `warn` | Visible in output, but CI passes |
| `block` | CI fails (non-zero exit code) |

You only need to define actions for categories you care about. Omitted categories default to no action.

## Rules

Rules filter which findings the policy applies to. Multiple rules use OR logic by default (any match includes the finding).

### Rule Structure

```json
{ "field": "FieldName", "type": "operator", "value": "match-value" }
```

### Rule Operators

| Operator | Description |
|----------|-------------|
| `eq` | Equals (exact match) |
| `ne` | Not equals |
| `lt` | Less than (semantic version aware) |
| `gt` | Greater than (semantic version aware) |
| `le` | Less than or equal |
| `ge` | Greater than or equal |
| `contains` | String contains substring |
| `not_contains` | String does not contain substring |
| `hasPrefix` | String starts with |
| `hasSuffix` | String ends with |
| `regex` | Regular expression match |

Field names are **case-insensitive**: `Severity`, `severity`, and `SEVERITY` all work.

### AND vs OR Logic

```json
{
  "operator": "OR",
  "rules": [
    { "field": "Severity", "type": "eq", "value": "CRITICAL" },
    { "field": "Severity", "type": "eq", "value": "HIGH" }
  ]
}
```
Matches CRITICAL **or** HIGH severity.

```json
{
  "operator": "AND",
  "rules": [
    { "field": "Severity", "type": "eq", "value": "CRITICAL" },
    { "field": "PkgName", "type": "eq", "value": "lodash" }
  ]
}
```
Matches only CRITICAL severity **and** package name is lodash.

## Output Configuration

Each policy can have multiple outputs:

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
  "comment": "Please fix before merging"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `format` | Yes | `markdown`, `html`, or `json` |
| `destination` | Yes | Where to send output (see below) |
| `template` | No | `table` or `text` (ignored for JSON) |
| `fields` | No | Which fields to include |
| `changes` | No | Filter to specific change categories |
| `group_by` | No | Group results by these fields |
| `combined` | No | Merge with other outputs at same destination |
| `collapse` | No | Make section collapsible (markdown/html) |
| `title` | No | Custom section title |
| `comment` | No | Text shown below title |

See [Outputs](./outputs.md) for full details on formats, destinations, and combining.

## Default Configuration

When no `.codeward.json` exists, Codeward uses these defaults:

```json
{
  "global": {
    "dependency_tree": false
  },
  "vulnerability": [{
    "name": "Vulnerability issues",
    "actions": {
      "new": "block",
      "existing": "warn",
      "removed": "info"
    },
    "rules": [
      { "field": "Severity", "type": "eq", "value": "CRITICAL" },
      { "field": "Severity", "type": "eq", "value": "HIGH" },
      { "field": "Severity", "type": "eq", "value": "MEDIUM" }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "template": "table",
      "fields": ["PkgID", "VulnerabilityID", "InstalledVersion", "FixedVersion", "Severity"],
      "changes": ["new", "removed", "existing"],
      "collapse": true,
      "title": "[codeward.io] Vulnerability Issues",
      "comment": "Please contact the security team to review the vulnerability issues."
    }]
  }],
  "license": [{
    "name": "License issues",
    "actions": {
      "new": "block",
      "existing": "warn",
      "removed": "info"
    },
    "rules": [
      { "field": "Severity", "type": "eq", "value": "UNKNOWN" },
      { "field": "Severity", "type": "eq", "value": "CRITICAL" },
      { "field": "Severity", "type": "eq", "value": "HIGH" }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "template": "table",
      "fields": ["Severity", "PkgName", "Category", "Name"],
      "changes": ["existing", "new", "removed"],
      "collapse": true,
      "comment": "Please contact the legal team to review the license issues."
    }]
  }]
}
```

## Starter Configurations

### Minimal: Block Critical Only

```json
{
  "vulnerability": [{
    "name": "block-critical",
    "actions": { "new": "block" },
    "rules": [
      { "field": "Severity", "type": "eq", "value": "CRITICAL" }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
      "changes": ["new"]
    }]
  }]
}
```

### Progressive: Observe First, Then Block

```json
{
  "vulnerability": [
    {
      "name": "critical-block",
      "actions": { "new": "block", "existing": "warn" },
      "rules": [{ "field": "Severity", "type": "eq", "value": "CRITICAL" }],
      "outputs": [{
        "format": "markdown",
        "destination": "git:pr",
        "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
        "changes": ["new"],
        "collapse": true
      }]
    },
    {
      "name": "high-observe",
      "actions": { "new": "warn" },
      "rules": [{ "field": "Severity", "type": "eq", "value": "HIGH" }],
      "outputs": [{
        "format": "markdown",
        "destination": "git:pr",
        "fields": ["VulnerabilityID", "PkgName", "Severity"],
        "changes": ["new"],
        "collapse": true
      }]
    }
  ]
}
```

### Full: Vulnerabilities + Licenses + Packages

```json
{
  "vulnerability": [{
    "name": "crit-high-block",
    "actions": { "new": "block", "existing": "warn" },
    "rules": [
      { "field": "Severity", "type": "eq", "value": "CRITICAL" },
      { "field": "Severity", "type": "eq", "value": "HIGH" }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
      "changes": ["new"]
    }]
  }],
  "license": [{
    "name": "gpl-block",
    "actions": { "new": "block" },
    "rules": [{ "field": "Category", "type": "eq", "value": "Copyleft" }],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "fields": ["Name", "Category", "Severity", "PkgName"],
      "changes": ["new"]
    }]
  }],
  "package": [{
    "name": "new-packages",
    "actions": { "new": "info" },
    "rules": [],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "fields": ["Name", "Version"],
      "changes": ["new"]
    }]
  }]
}
```

## Progressive Enforcement

Start with observation, then gradually tighten:

| Phase | Approach |
|-------|----------|
| 1. Observe | All actions set to `info` — see what would be flagged |
| 2. Warn | Change `new` to `warn` — visible but non-blocking |
| 3. Block Critical | Set `new: block` for CRITICAL only |
| 4. Expand | Add HIGH to blocking, then more categories |
| 5. Steady State | Block what matters, warn on the rest |

**Tip**: Use separate policies for different severity levels so you can promote them independently.

## Common Mistakes

| Problem | Cause | Fix |
|---------|-------|-----|
| Rules ignored | Used nested object format | Use array: `"rules": [{ "field": "...", "type": "...", "value": "..." }]` |
| Everything shows as "new" | Missing baseline scan | Mount `/main` and set `CI_EVENT=pr` |
| Template error with JSON | Set `template` for JSON format | Remove `template` — JSON doesn't use templates |
| Mixed formats in combined | Combined outputs mix JSON with markdown | Use same format for all combined outputs |
| Invalid field error | Field not available for policy type | Check allowed fields for each policy type |

See [Policies](./policies.md) for allowed fields per policy type.
