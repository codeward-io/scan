# Policies

Codeward supports five policy types. Each scans for different concerns but shares the same configuration pattern for actions, rules, and outputs.

## Policy Types Overview

| Type | What It Scans | Common Use Cases |
|------|---------------|------------------|
| **vulnerability** | CVEs in dependencies | Block critical/high severity, track fixes |
| **license** | License names and categories | Block copyleft, flag unknown licenses |
| **package** | Dependency additions/removals | Review new packages, track version changes |
| **validation** | File contents and filesystem | Enforce config standards, require files |
| **pr_validation** | PR metadata and file changes | Limit PR size, enforce naming conventions |

---

## Vulnerability Policies

Detect security vulnerabilities (CVEs) in your dependencies using embedded Trivy scanner.

### Allowed Fields

| Field | Description |
|-------|-------------|
| `VulnerabilityID` | CVE identifier (e.g., CVE-2024-1234) |
| `PkgID` | Package identifier |
| `PkgName` | Package name |
| `InstalledVersion` | Currently installed version |
| `FixedVersion` | Version that fixes the vulnerability |
| `Status` | Fix status (e.g., "fixed", "affected") |
| `Severity` | CRITICAL, HIGH, MEDIUM, LOW, UNKNOWN |
| `Title` | Vulnerability title |
| `Description` | Detailed description |
| `PrimaryURL` | Primary reference URL |
| `PublishedDate` | When vulnerability was published |
| `CweIDs` | CWE identifiers (array) |
| `Relationship` | `direct` or `indirect` (requires `dependency_tree: true`) |
| `Parents` | Parent packages (requires `dependency_tree: true`) |
| `Children` | Child packages (requires `dependency_tree: true`) |

### Examples

**Block new critical vulnerabilities:**
```json
{
  "vulnerability": [{
    "name": "block-critical",
    "actions": { "new": "block", "existing": "warn" },
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

**Block critical and high:**
```json
{
  "rules": [
    { "field": "Severity", "type": "eq", "value": "CRITICAL" },
    { "field": "Severity", "type": "eq", "value": "HIGH" }
  ]
}
```

**Only direct dependencies (block transitive noise):**
```json
{
  "global": { "dependency_tree": true },
  "vulnerability": [{
    "name": "direct-only",
    "actions": { "new": "block" },
    "operator": "AND",
    "rules": [
      { "field": "Severity", "type": "eq", "value": "CRITICAL" },
      { "field": "Relationship", "type": "eq", "value": "direct" }
    ],
    "outputs": [{ "format": "markdown", "destination": "git:pr" }]
  }]
}
```

**Exclude a specific CVE:**
```json
{
  "rules": [
    { "field": "Severity", "type": "eq", "value": "CRITICAL" },
    { "field": "VulnerabilityID", "type": "ne", "value": "CVE-2024-ACCEPTED" }
  ]
}
```

---

## License Policies

Detect license issues for compliance. Trivy categorizes licenses by risk level.

### Allowed Fields

| Field | Description |
|-------|-------------|
| `Name` | License name (e.g., MIT, GPL-3.0) |
| `Category` | License category (e.g., Permissive, Copyleft, Forbidden) |
| `Severity` | Risk level: CRITICAL, HIGH, MEDIUM, LOW, UNKNOWN |
| `PkgName` | Package using this license |
| `Relationship` | `direct` or `indirect` (requires `dependency_tree: true`) |
| `Parents` | Parent packages |
| `Children` | Child packages |

### Examples

**Block copyleft licenses:**
```json
{
  "license": [{
    "name": "no-copyleft",
    "actions": { "new": "block", "existing": "warn" },
    "rules": [
      { "field": "Category", "type": "eq", "value": "Copyleft" }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "fields": ["Name", "Category", "PkgName"],
      "changes": ["new"]
    }]
  }]
}
```

**Block GPL by name:**
```json
{
  "rules": [
    { "field": "Name", "type": "contains", "value": "GPL" }
  ]
}
```

**Flag unknown licenses:**
```json
{
  "rules": [
    { "field": "Severity", "type": "eq", "value": "UNKNOWN" }
  ]
}
```

---

## Package Policies

Track dependency changes: additions, removals, and version updates.

### Allowed Fields

| Field | Description |
|-------|-------------|
| `ID` | Package identifier |
| `Name` | Package name |
| `Version` | Package version |
| `Relationship` | `direct` or `indirect` (requires `dependency_tree: true`) |
| `Parents` | Parent packages |
| `Children` | Child packages |

### Examples

**Track new packages:**
```json
{
  "package": [{
    "name": "new-packages",
    "actions": { "new": "warn" },
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

**Only direct dependencies:**
```json
{
  "global": { "dependency_tree": true },
  "package": [{
    "name": "new-direct",
    "actions": { "new": "warn" },
    "rules": [
      { "field": "Relationship", "type": "eq", "value": "direct" }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "fields": ["Name", "Version", "Relationship"],
      "changes": ["new"]
    }]
  }]
}
```

**Export full inventory:**
```json
{
  "package": [{
    "name": "inventory",
    "actions": { "new": "info", "existing": "info", "changed": "info", "removed": "info" },
    "rules": [],
    "outputs": [{
      "format": "json",
      "destination": "file:/results/packages.json"
    }]
  }]
}
```

---

## Validation Policies

Assert rules against file contents or filesystem structure. Great for enforcing standards in Dockerfiles, CI configs, package.json, etc.

### Additional Required Fields

Validation policies require extra fields:

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | `text`, `json`, `yaml`, `yml`, or `filesystem` |
| `path` | Yes | File path or glob pattern to check |

### Validation Types and Rule Keys

| Type | Rule Keys | Available Operators |
|------|-----------|---------------------|
| `text` | `type`, `value` | `regex`, `contains`, `not_contains`, `hasPrefix`, `hasSuffix`, `eq`, `ne` |
| `json` | `type`, `key`, `value` | `eq`, `ne`, `lt`, `gt`, `le`, `ge`, `contains`, `not_contains`, `regex`, `exists`, `not_exists` |
| `yaml` / `yml` | `type`, `key`, `value` | Same as `json` |
| `filesystem` | `type`, `path` | `exists`, `not_exists` |

### Output Fields

Validation policies use different output fields:

| Field | Description |
|-------|-------------|
| `Key` | The rule key checked |
| `Type` | The rule type used |
| `Value` | The expected value |
| `Reason` | Explanation of pass/fail |
| `Passing` | Boolean result |
| `Path` | File path checked |

### Examples

**Require SECURITY.md file:**
```json
{
  "validation": [{
    "name": "require-security-md",
    "type": "filesystem",
    "path": ".",
    "actions": { "new": "block" },
    "rules": [
      { "type": "exists", "path": "SECURITY.md" }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "fields": ["Path", "Reason", "Passing"]
    }]
  }]
}
```

**Dockerfile must have USER instruction:**
```json
{
  "validation": [{
    "name": "dockerfile-nonroot",
    "type": "text",
    "path": "Dockerfile",
    "actions": { "new": "block", "changed": "block" },
    "rules": [
      { "type": "contains", "value": "USER" }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "fields": ["Key", "Reason", "Passing"]
    }]
  }]
}
```

**package.json must have test script:**
```json
{
  "validation": [{
    "name": "require-test-script",
    "type": "json",
    "path": "package.json",
    "actions": { "new": "warn" },
    "rules": [
      { "type": "exists", "key": "scripts.test" }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr"
    }]
  }]
}
```

**CI workflow must have minimal permissions:**
```json
{
  "validation": [{
    "name": "workflow-permissions",
    "type": "yaml",
    "path": ".github/workflows/*.yml",
    "actions": { "new": "block", "changed": "block" },
    "rules": [
      { "type": "eq", "key": "permissions.contents", "value": "read" }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr"
    }]
  }]
}
```

---

## PR Validation Policies

Assert rules against pull request metadata and file changes. Only runs when `CI_EVENT=pr`.

### PR-Level Keys

| Key | Description |
|-----|-------------|
| `title` | PR title |
| `state` | PR state (open, closed, merged) |
| `commits` | Number of commits |
| `changed_files` | Number of files changed |
| `total_added` | Total lines added |
| `total_removed` | Total lines removed |
| `user.login` | PR author username |
| `user.type` | Author type (User, Bot, Organization) |
| `comments.count` | Number of comments |
| `requested_reviewers.count` | Number of requested reviewers |

### File-Level Keys

Use `files.*` prefix to check individual file changes:

| Key | Description |
|-----|-------------|
| `files.filename` | File path |
| `files.status` | File status: added, removed, modified, renamed |
| `files.additions` | Lines added in this file |
| `files.deletions` | Lines deleted |
| `files.changes` | Total changes (additions + deletions) |
| `files.patch` | Unified diff content |

### File Filtering

| Field | Description |
|-------|-------------|
| `file_pattern` | Regex pattern to match file paths |
| `file_status` | Filter by status: `new`, `changed`, `removed`, `renamed`, or `*` |
| `action` | Per-rule action: `info`, `warn`, or `block` |

### Examples

**Limit PR size:**
```json
{
  "pr_validation": [{
    "name": "pr-size-limit",
    "rules": [
      { "type": "lt", "key": "changed_files", "value": "20", "action": "warn" },
      { "type": "lt", "key": "total_added", "value": "500", "action": "warn" }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr"
    }]
  }]
}
```

**Require conventional commit titles:**
```json
{
  "pr_validation": [{
    "name": "pr-title-convention",
    "rules": [
      { "type": "regex", "key": "title", "value": "^(feat|fix|docs|refactor|test|chore):", "action": "block" }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr"
    }]
  }]
}
```

**No changes to sensitive files:**
```json
{
  "pr_validation": [{
    "name": "protect-secrets",
    "rules": [
      { 
        "type": "not_exists", 
        "key": "files.filename",
        "file_pattern": ".*\\.env.*",
        "action": "block"
      }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr"
    }]
  }]
}
```

**Check patch content for secrets:**
```json
{
  "pr_validation": [{
    "name": "no-hardcoded-secrets",
    "rules": [
      { 
        "type": "not_contains",
        "key": "files.patch",
        "value": "password=",
        "file_status": "*",
        "action": "block"
      }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr"
    }]
  }]
}
```

---

## Rule Operators Reference

Available for all policy types:

| Operator | Description | Example |
|----------|-------------|---------|
| `eq` | Equals | `"CRITICAL"` |
| `ne` | Not equals | Exclude specific values |
| `lt` | Less than | Version or numeric comparison |
| `gt` | Greater than | Version or numeric comparison |
| `le` | Less than or equal | |
| `ge` | Greater than or equal | |
| `contains` | Substring match | `"GPL"` in license name |
| `not_contains` | Substring not present | |
| `hasPrefix` | Starts with | `"@types/"` packages |
| `hasSuffix` | Ends with | `.test.js` files |
| `regex` | Regular expression | `"^CVE-2024-.*"` |

Additional for validation/filesystem:
| `exists` | Key or file exists | |
| `not_exists` | Key or file does not exist | |

**Version-aware comparisons**: `lt`, `gt`, `le`, `ge` automatically detect semantic versions and compare correctly (e.g., `1.10.0 > 1.9.0`).

---

## Best Practices

1. **Start with warnings**: Use `warn` for new policies, promote to `block` after validation
2. **Focus on `new`**: Only block `new` findings — let existing issues be tracked separately
3. **Keep policies small**: One intent per policy makes debugging easier
4. **Use meaningful names**: Policy names appear in reports
5. **Add JSON outputs**: Export to files for dashboards and automation
6. **Enable dependency_tree**: When filtering by `Relationship`, `Parents`, or `Children`
