# Configuration Overview

Codeward uses a flexible, JSON-based configuration system that allows you to define security policies, customize scanning behavior, and control output generation.

## Configuration Architecture

### Two-File Configuration System

Codeward uses two separate configuration files:

1. **Main Configuration (`config.json`)** - Defines security policies and scanning rules
2. **Private Configuration** - Contains environment-specific settings like repository paths

```
┌─────────────────────┐    ┌──────────────────────┐
│   config.json       │    │  private_config.json │
│  (Version Control)  │    │   (Environment)      │
├─────────────────────┤    ├──────────────────────┤
│ • Policies          │    │ • Repository paths   │
│ • Rules             │ +  │ • Output paths       │
│ • Output formats    │    │ • GitHub settings    │
│ • Actions           │    │ • CI/CD config       │
└─────────────────────┘    └──────────────────────┘
            │                          │
            └──────────┬─────────────────┘
                       ▼
            ┌─────────────────────┐
            │     Codeward        │
            │     Engine          │
            └─────────────────────┘
```

### Configuration Types

#### Main Configuration (`config.json`)
- **Purpose**: Define security policies, rules, and output formats
- **Location**: Repository root or specified via `CONFIG_PATH` environment variable
- **Visibility**: Version controlled and shared across team
- **Contents**: Vulnerability, license, package, and validation policies

#### Private Configuration
- **Purpose**: Environment-specific settings and runtime parameters
- **Location**: Separate file specified via `PRIVATE_CONFIG_PATH` environment variable
- **Visibility**: Not version controlled, deployment-specific
- **Contents**: Repository paths, GitHub tokens, CI/CD settings

## Main Configuration Structure

### Complete Schema

```json
{
  "global": {
    "dependency_tree": false
  },
  "vulnerability": [
    {
      "name": "Policy name",
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
        ],
        "pkgName": [
          {"type": "contains", "value": "example"}
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
          "comment": "Review these security issues",
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
        "existing": "info"
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
      "name": "Package changes",
      "disabled": false,
      "actions": {
        "new": "info",
        "removed": "warn"
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
      "name": "Dockerfile validation",
      "path": "**/Dockerfile",
      "type": "text",
      "disabled": false,
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

### Global Settings

The `global` section contains settings that apply to all policies:

```json
{
  "global": {
    "dependency_tree": false  // Enable/disable dependency relationship analysis
  }
}
```

**Options:**
- `dependency_tree` (boolean): When `true`, Codeward will analyze and include dependency relationships in package reports

### Policy Structure

All policy types (vulnerability, license, package, validation) share a common structure:

#### Required Fields
- `name` (string): Human-readable policy name for identification

#### Optional Fields  
- `disabled` (boolean): Set to `true` to disable this policy (default: `false`)
- `actions` (object): Define actions for different change types
- `rules` (object): Filter criteria for the policy
- `outputs` (array): Output configurations for this policy

#### Actions Configuration

Actions determine what Codeward does when it finds issues:

```json
{
  "actions": {
    "new": "block",      // Items introduced in the branch
    "existing": "warn",  // Items present in both main and branch
    "removed": "info",   // Items removed from main in the branch
    "changed": "info"    // Items that changed between main and branch
  }
}
```

**Action Types:**
- `info`: Log information only
- `warn`: Log warning but continue execution
- `block`: Log error and exit with non-zero code (fails CI/CD)

#### Rules Configuration

Rules filter what items the policy applies to:

```json
{
  "rules": {
    "severity": [
      {"type": "eq", "value": "CRITICAL"},
      {"type": "ne", "value": "LOW"}
    ],
    "pkgName": [
      {"type": "contains", "value": "security"},
      {"type": "regex", "value": "^@company/.*"}
    ]
  }
}
```

**Rule Operators:**
- `eq`: Equals
- `ne`: Not equals  
- `lt`: Less than
- `gt`: Greater than
- `le`: Less than or equal
- `ge`: Greater than or equal
- `contains`: Contains substring
- `not_contains`: Does not contain substring
- `hasPrefix`: Starts with
- `hasSuffix`: Ends with
- `regex`: Regular expression match
- `exists`: Field exists (validation only)
- `not_exists`: Field does not exist (validation only)

#### Output Configuration

Outputs define how and where results are reported:

```json
{
  "outputs": [
    {
      "format": "markdown",           // Output format
      "template": "table",            // Template type
      "destination": "git:pr",        // Where to send output
      "fields": ["Field1", "Field2"], // Which fields to include
      "changes": ["new", "existing"], // Which change types to include
      "group_by": ["PkgName"],        // Group results by these fields
      "title": "Report Title",        // Title for the output
      "comment": "Additional info",   // Additional context
      "collapse": true                // Collapse output in GitHub
    }
  ]
}
```

**Format Options:**
- `markdown`: Markdown format
- `html`: HTML format  
- `json`: JSON format

**Template Options:**
- `table`: Traditional table format (default)
- `text`: Professional text format with emoji indicators

**Destination Options:**
- `log:stdout`: Print to console output
- `log:stderr`: Print to error output
- `file:/path/to/file`: Save to file
- `git:pr`: Post as GitHub PR comment
- `git:issue`: Create GitHub issue

**Available Fields by Policy Type:**

*Vulnerability:*
- `VulnerabilityID`, `PkgID`, `PkgName`, `InstalledVersion`, `FixedVersion`, `Status`, `Severity`

*License:*
- `Severity`, `Category`, `PkgName`, `Name`

*Package:*  
- `ID`, `Name`, `Version`, `Relationship`, `Children`, `Parents`, `Targets`

*Validation:*
- `key`, `type`, `value`, `reason`, `passing`, `path`

## Policy Types

### Vulnerability Policies

Scan for security vulnerabilities using Trivy scanner:

```json
{
  "vulnerability": [
    {
      "name": "Critical security issues",
      "disabled": false,
      "actions": {
        "new": "block",
        "existing": "warn"
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
          "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"]
        }
      ]
    }
  ]
}
```

### License Policies

Monitor license compliance across dependencies:

```json
{
  "license": [
    {
      "name": "License compliance check",
      "disabled": false,
      "actions": {
        "new": "warn"
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
  ]
}
```

### Package Policies

Track dependency changes between repository states:

```json
{
  "package": [
    {
      "name": "Dependency changes",
      "disabled": false,
      "actions": {
        "new": "info",
        "removed": "warn"
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
  ]
}
```

### Validation Policies

Custom file content validation:

```json
{
  "validation": [
    {
      "name": "Dockerfile security check",
      "path": "**/Dockerfile",
      "type": "text",
      "disabled": false,
      "actions": {
        "new": "block"
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

**Validation Types:**
- `text`: Plain text files
- `json`: JSON files  
- `yaml`/`yml`: YAML files
- `filesystem`: File/directory existence checks

## Private Configuration

The private configuration contains environment-specific settings:

```json
{
  "main_path": "/path/to/main/branch",
  "branch_path": "/path/to/feature/branch"
}
```

For GitHub integration, additional fields may be required based on your CI/CD setup.

## Environment Variables

Codeward uses environment variables for configuration:

- `CONFIG_PATH`: Path to main configuration file
- `PRIVATE_CONFIG_PATH`: Path to private configuration file
- `GITHUB_TOKEN`: GitHub API token (for git: destinations)
- `GITHUB_OWNER`: Repository owner
- `GITHUB_REPO`: Repository name
- `GITHUB_PR_NR`: Pull request number
- `CI_EVENT`: Event type (pr, main)
- `CI`: CI environment flag

## Default Configurations

Codeward includes embedded default configurations when no config files are provided:

- **CI with PR event**: Compares main vs branch
- **CI with main event**: Scans main branch only
- **Local development**: Basic vulnerability and license scanning

## Configuration Validation

Codeward validates all configuration on startup:

- Required fields presence
- Valid enum values (actions, formats, templates)
- Field compatibility (rules match policy types)
- Output destination format
- Template and format combinations

Validation errors include detailed field paths:
```
vulnerability[0]: missing required field 'name'
license[1].outputs[0]: invalid format 'txt', must be 'markdown', 'html', or 'json'
```

## Next Steps

- **[Main Configuration](./main-config.md)** - Detailed policy configuration
- **[Policy System](../concepts/policy-system.md)** - Understanding how policies work
