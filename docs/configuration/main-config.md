# Main Configuration

The main configuration file (`config.json`) defines the security policies and scanning behavior for Codeward. This file contains policy definitions for vulnerability, license, package, and custom validation scanning.

## Configuration File Location

The main configuration file can be:
- Named `config.json` in your repository root
- Specified via the `CONFIG_PATH` environment variable
- Auto-loaded from embedded defaults if no file is provided

## Complete Configuration Example

```json
{
  "global": {
    "dependency_tree": false
  },
  "vulnerability": [
    {
      "name": "Critical vulnerability blocking",
      "disabled": false,
      "actions": {
        "new": "block",
        "existing": "warn",
        "removed": "info",
        "changed": "warn"
      },
      "rules": {
        "severity": [
          {"type": "eq", "value": "CRITICAL"},
          {"type": "eq", "value": "HIGH"}
        ]
      },
      "outputs": [
        {
          "format": "markdown",
          "template": "table",
          "destination": "git:pr",
          "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
          "changes": ["new", "existing"],
          "title": "Security Vulnerabilities",
          "comment": "Please review these security issues",
          "collapse": true
        }
      ]
    }
  ],
  "license": [
    {
      "name": "License compliance",
      "disabled": false,
      "actions": {
        "new": "warn",
        "existing": "info",
        "removed": "info"
      },
      "rules": {
        "severity": [
          {"type": "eq", "value": "CRITICAL"},
          {"type": "eq", "value": "HIGH"}
        ]
      },
      "outputs": [
        {
          "format": "markdown",
          "template": "table",
          "destination": "git:pr",
          "fields": ["Name", "Category", "PkgName"]
        }
      ]
    }
  ],
  "package": [
    {
      "name": "Package change tracking",
      "disabled": false,
      "actions": {
        "new": "info",
        "existing": "info",
        "removed": "warn",
        "changed": "info"
      },
      "outputs": [
        {
          "format": "markdown",
          "template": "table",
          "destination": "log:stdout",
          "fields": ["Name", "Version", "Relationship"]
        }
      ]
    }
  ],
  "validation": [
    {
      "name": "Dockerfile security",
      "disabled": false,
      "path": "**/Dockerfile",
      "type": "text",
      "actions": {
        "new": "block",
        "existing": "warn"
      },
      "rules": [
        {
          "type": "contains",
          "value": "USER"
        }
      ],
      "outputs": [
        {
          "format": "markdown",
          "template": "table",
          "destination": "git:pr",
          "fields": ["key", "type", "value", "reason", "passing", "path"]
        }
      ]
    }
  ]
}
```

## Configuration Sections

### Global Settings

The `global` section contains settings that apply to all scanning policies:

```json
{
  "global": {
    "dependency_tree": false  // Enable/disable dependency relationship analysis
  }
}
```

**Available Options:**
- `dependency_tree` (boolean): When `true`, Codeward analyzes and includes dependency relationships in package reports

### Policy Types

Codeward supports four types of policies, each with a consistent structure:

#### Required Fields (All Policies)
- `name` (string): Human-readable policy identifier

#### Optional Fields (All Policies)
- `disabled` (boolean): Set to `true` to disable the policy (default: `false`)
- `actions` (object): Define actions for different change types
- `rules` (object): Filter criteria for the policy  
- `outputs` (array): Output configurations for reports

### Vulnerability Policies

Scan for security vulnerabilities using Trivy scanner:

```json
{
  "vulnerability": [
    {
      "name": "Critical vulnerabilities",
      "disabled": false,
      "actions": {
        "new": "block",        // Block on new critical issues
        "existing": "warn",    // Warn on existing issues
        "removed": "info",     // Info when issues are fixed
        "changed": "warn"      // Warn on changed issues
      },
      "rules": {
        "severity": [
          {"type": "eq", "value": "CRITICAL"},
          {"type": "eq", "value": "HIGH"}
        ],
        "pkgName": [
          {"type": "contains", "value": "security"}
        ],
        "vulnerabilityID": [
          {"type": "regex", "value": "CVE-2023-.*"}
        ]
      },
      "outputs": [
        {
          "format": "markdown",
          "template": "table",
          "destination": "git:pr",
          "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
          "changes": ["new", "existing"],
          "group_by": ["PkgName", "Severity"],
          "title": "Security Vulnerabilities",
          "comment": "Please review and fix these security issues",
          "collapse": true
        }
      ]
    }
  ]
}
```

**Available Rule Fields:**
- `severity`: CRITICAL, HIGH, MEDIUM, LOW
- `pkgName`: Package name patterns
- `vulnerabilityID`: CVE IDs or patterns
- `pkgID`: Package ID patterns
- `status`: Vulnerability status

### License Policies

Monitor license compliance across dependencies:

```json
{
  "license": [
    {
      "name": "License compliance",
      "disabled": false,
      "actions": {
        "new": "warn",
        "existing": "info"
      },
      "rules": {
        "severity": [
          {"type": "eq", "value": "CRITICAL"},
          {"type": "eq", "value": "HIGH"}
        ],
        "name": [
          {"type": "contains", "value": "GPL"}
        ]
      },
      "outputs": [
        {
          "format": "markdown",
          "template": "table",
          "destination": "git:pr",
          "fields": ["Name", "Category", "PkgName", "Severity"],
          "group_by": ["Category"]
        }
      ]
    }
  ]
}
```

**Available Rule Fields:**
- `severity`: License risk level (CRITICAL, HIGH, MEDIUM, LOW, UNKNOWN)
- `name`: License name patterns
- `pkgName`: Package name patterns

### Package Policies

Track dependency changes between repository states:

```json
{
  "package": [
    {
      "name": "Dependency tracking",
      "disabled": false,
      "actions": {
        "new": "info",      // Info on new dependencies
        "removed": "warn",  // Warn on removed dependencies
        "changed": "info"   // Info on version changes
      },
      "rules": {
        "name": [
          {"type": "hasPrefix", "value": "@types/"}
        ],
        "version": [
          {"type": "contains", "value": "beta"}
        ]
      },
      "outputs": [
        {
          "format": "markdown",
          "template": "table",
          "destination": "log:stdout",
          "fields": ["Name", "Version", "Relationship"],
          "changes": ["new", "removed"]
        }
      ]
    }
  ]
}
```

**Available Rule Fields:**
- `name`: Package name patterns
- `version`: Version patterns
- `relationship`: Dependency relationship type

### Validation Policies

Custom file content validation:

```json
{
  "validation": [
    {
      "name": "Dockerfile security",
      "disabled": false,
      "path": "**/Dockerfile",
      "type": "text",
      "actions": {
        "new": "block",
        "existing": "warn"
      },
      "rules": [
        {
          "type": "contains",
          "value": "USER"
        },
        {
          "type": "not_contains", 
          "value": "root"
        }
      ],
      "outputs": [
        {
          "format": "markdown",
          "template": "table",
          "destination": "git:pr",
          "fields": ["key", "type", "value", "reason", "passing", "path"]
        }
      ]
    }
  ]
}
```

**Required Fields:**
- `path`: File path pattern (supports glob patterns)
- `type`: File type (text, json, yaml, yml, filesystem)

**Validation Types:**
- `text`: Plain text files with regex/contains validation
- `json`: JSON files with key-based validation
- `yaml`/`yml`: YAML files (converted to JSON for validation)
- `filesystem`: File/directory existence checks

**Available Rule Types:**
- `eq`, `ne`: Equals/not equals
- `lt`, `gt`, `le`, `ge`: Numeric comparisons
- `contains`, `not_contains`: Substring matching
- `hasPrefix`, `hasSuffix`: String prefix/suffix
- `regex`: Regular expression matching
- `exists`, `not_exists`: Field/file existence

## Output Configuration

### Output Structure

Each policy can have multiple outputs with different destinations and formats:

```json
{
  "outputs": [
    {
      "format": "markdown",           // Output format: markdown, html, json
      "template": "table",            // Template: table or text (ignored for json)
      "destination": "git:pr",        // Where to send the output
      "fields": ["Field1", "Field2"], // Which fields to include
      "changes": ["new", "existing"], // Which change types to report
      "group_by": ["PkgName"],        // Fields to group results by
      "title": "Report Title",        // Title for the output
      "comment": "Additional info",   // Additional context
      "collapse": true                // Collapse sections in GitHub
    }
  ]
}
```

### Format and Template Options

**Formats:**
- `markdown`: Markdown format for GitHub/documentation
- `html`: HTML format for web display
- `json`: JSON format for API integration

**Templates:** (ignored for JSON format)
- `table`: Traditional table format (default, backward compatible)
- `text`: Professional text format with emoji indicators and improved readability

### Destination Options

**Local Output:**
- `log:stdout`: Print to console output
- `log:stderr`: Print to error output
- `file:/path/to/file.md`: Save to local file

**GitHub Integration:**
- `git:pr`: Post as GitHub PR comment
- `git:issue`: Create GitHub issue

### Field Selection

Specify which fields to include in reports:

**Vulnerability Fields:**
- `VulnerabilityID`, `PkgID`, `PkgName`, `InstalledVersion`, `FixedVersion`, `Status`, `Severity`

**License Fields:**
- `Severity`, `Category`, `PkgName`, `Name`

**Package Fields:**
- `ID`, `Name`, `Version`, `Relationship`, `Children`, `Parents`, `Targets`

**Validation Fields:**
- `key`, `type`, `value`, `reason`, `passing`, `path`

### Change Type Filtering

Control which types of changes to report:

```json
{
  "changes": ["new", "existing", "removed", "changed"]
}
```

**Change Types:**
- `new`: Items introduced in the feature branch
- `existing`: Items present in both main and feature branch
- `removed`: Items removed from main in the feature branch
- `changed`: Items that changed between main and feature branch

### Grouping Results

Group results by any field combination:

```json
{
  "group_by": ["PkgName", "Severity"]  // Group by package and severity
}
```

**Grouping Behavior:**
- When `Severity` field is not included in grouping, shows highest severity from grouped items
- Empty array means no grouping (shows individual items)
- Multiple fields create nested groupings

## Policy Management

### Disabling Policies

Individual policies can be disabled without removing them:

```json
{
  "vulnerability": [
    {
      "name": "Disabled policy",
      "disabled": true,  // This policy will be skipped
      "actions": {
        "new": "block"
      }
      // ... rest of configuration
    }
  ]
}
```

When disabled, Codeward logs the policy name for transparency.

### Action Types

Define what happens when issues are found:

```json
{
  "actions": {
    "new": "block",      // Exit with error code (fails CI/CD)
    "existing": "warn",  // Log warning but continue
    "removed": "info",   // Log information only
    "changed": "warn"    // Log warning but continue
  }
}
```

**Action Behaviors:**
- `info`: Logs information, continues execution
- `warn`: Logs warning, continues execution
- `block`: Logs error and exits with non-zero code (fails CI/CD)

## Configuration Validation

Codeward performs comprehensive validation on startup:

### Validation Checks
- JSON syntax and structure
- Required field presence
- Valid enumeration values (actions, formats, templates)
- Field compatibility with policy types
- Output destination format validation
- Rule operator compatibility

### Common Validation Errors

**Missing Required Fields:**
```
vulnerability[0]: missing required field 'name'
```

**Invalid Action Values:**
```
vulnerability[0].actions.new: invalid action 'fail', must be 'info', 'warn', or 'block'
```

**Invalid Format/Template:**
```
vulnerability[0].outputs[0]: invalid format 'txt', must be 'markdown', 'html', or 'json'
vulnerability[0].outputs[0]: invalid template 'custom', must be 'table' or 'text'
```

**Field Compatibility:**
```
license[0].outputs[0]: field 'VulnerabilityID' is not valid for license outputs
```

## Environment Variables

**Configuration Loading:**
- `CONFIG_PATH`: Path to main configuration file
- `PRIVATE_CONFIG_PATH`: Path to private configuration file

**GitHub Integration:** (required for `git:pr` and `git:issue` destinations)
- `GITHUB_TOKEN`: GitHub API token
- `GITHUB_OWNER`: Repository owner
- `GITHUB_REPO`: Repository name
- `GITHUB_PR_NR`: Pull request number
- `CI_EVENT`: Event type (`pr` or `main`)

## Default Configuration Behavior

If no `CONFIG_PATH` is provided, Codeward uses embedded defaults:

**CI Environment:**
- `CI_EVENT=pr`: Compares main vs branch with GitHub integration
- `CI_EVENT=main`: Scans main branch only
- Auto-detects CI environment based on environment variables

**Local Development:**
- Basic vulnerability and license scanning
- Console output destinations
- Development-friendly actions (warn instead of block)

## Best Practices

### Configuration Organization
- Keep policies focused and well-named
- Use descriptive titles and comments for outputs
- Group related rules within policies
- Test configurations locally before deployment

### Performance Considerations
- Disable unnecessary policies for faster scans
- Use specific rules to reduce noise
- Consider output destinations based on team workflow
- Group results appropriately for readability

### Security Best Practices
- Use `block` action for critical security issues
- Monitor license compliance with appropriate actions
- Validate security-critical files (Dockerfiles, configs)
- Regular review and update of policies

## Next Steps

- **[Policy System](../concepts/policy-system.md)** - Understanding policy behavior
- **[Output Formats](../output/formats.md)** - Detailed output configuration
