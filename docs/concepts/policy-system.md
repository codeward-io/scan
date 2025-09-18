# Policy System

## Overview

Codeward uses a policy-based approach to define how security findings should be handled. Policies determine what actions to take when vulnerabilities, license issues, package changes, or validation failures are detected during diff analysis.

## Policy Types

Codeward supports four types of policies:

### Vulnerability Policies
Handle security vulnerabilities detected by Trivy scanner:
- **Severity-based rules**: Target specific vulnerability severities
- **Action mapping**: Different responses for new vs existing vulnerabilities
- **Output configuration**: Control where and how results are reported

### License Policies  
Monitor software license compliance:
- **License filtering**: Allow or block specific license types
- **Dependency tracking**: Monitor license changes in dependencies
- **Compliance reporting**: Generate license compliance reports

### Package Policies
Track package and dependency changes:
- **Dependency monitoring**: Track additions, removals, and updates
- **Version tracking**: Monitor package version changes
- **Change classification**: Identify significant package modifications

### Validation Policies
Custom code validation rules:
- **Pattern matching**: Search for specific patterns in code files
- **File targeting**: Apply rules to specific file types or paths
- **Custom messages**: Provide contextual feedback to developers

## Policy Structure

### Basic Policy Configuration
Each policy type is configured as an array in your `config.json`:

```json
{
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
        "severity": [{"type": "eq", "value": "CRITICAL"}]
      },
      "outputs": [
        {
          "format": "markdown",
          "template": "table",
          "destination": "git:pr"
        }
      ]
    }
  ]
}
```

### Policy Components

#### Name and Status
- **name**: Descriptive identifier for the policy
- **disabled**: Boolean flag to enable/disable the policy

#### Action Configuration
Different actions for each change type:
- **new**: Action for newly introduced issues
- **existing**: Action for pre-existing issues
- **removed**: Action for resolved issues  
- **changed**: Action for modified issues

#### Available Actions
- **block**: Fail the scan and prevent merge/deployment
- **warn**: Log a warning but allow continuation
- **info**: Informational logging only

#### Rules Configuration
Filter criteria for when the policy applies:
- **severity**: Target specific vulnerability severities
- **license**: Filter by license types
- **package**: Target specific packages
- **custom patterns**: For validation policies

#### Output Configuration
Control how results are reported:
- **format**: markdown, json, html
- **template**: table, text
- **destination**: git:pr, git:issue, file:path, log:stdout

## Policy Examples

### Vulnerability Policy Examples

#### Block Critical Vulnerabilities
```json
{
  "vulnerability": [
    {
      "name": "Block critical vulnerabilities",
      "disabled": false,
      "actions": {
        "new": "block",
        "existing": "warn",
        "removed": "info",
        "changed": "warn"
      },
      "rules": {
        "severity": [{"type": "eq", "value": "CRITICAL"}]
      },
      "outputs": [
        {
          "format": "markdown",
          "template": "table",
          "destination": "git:pr",
          "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
          "changes": ["new", "existing"]
        }
      ]
    }
  ]
}
```

#### Warn on High Severity Issues
```json
{
  "vulnerability": [
    {
      "name": "High severity vulnerability warnings",
      "disabled": false,
      "actions": {
        "new": "warn",
        "existing": "info",
        "removed": "info",
        "changed": "warn"
      },
      "rules": {
        "severity": [{"type": "eq", "value": "HIGH"}]
      },
      "outputs": [
        {
          "format": "json",
          "destination": "file:high-severity-report.json"
        }
      ]
    }
  ]
}
```

### License Policy Examples

#### Block GPL Licenses
```json
{
  "license": [
    {
      "name": "Block GPL licenses",
      "disabled": false,
      "actions": {
        "new": "block",
        "existing": "warn",
        "removed": "info",
        "changed": "block"
      },
      "rules": {
        "license": [{"type": "contains", "value": "GPL"}]
      },
      "outputs": [
        {
          "format": "markdown",
          "template": "text",
          "destination": "git:issue",
          "title": "License Compliance Issue Detected"
        }
      ]
    }
  ]
}
```

### Package Policy Examples

#### Track All Package Changes
```json
{
  "package": [
    {
      "name": "Package change tracking",
      "disabled": false,
      "actions": {
        "new": "info",
        "existing": "info",
        "removed": "info",
        "changed": "info"
      },
      "outputs": [
        {
          "format": "json",
          "destination": "file:package-changes.json"
        }
      ]
    }
  ]
}
```

### Validation Policy Examples

#### Detect Hardcoded Secrets
```json
{
  "validation": [
    {
      "name": "Hardcoded secrets detection",
      "disabled": false,
      "actions": {
        "new": "block",
        "existing": "warn",
        "removed": "info",
        "changed": "warn"
      },
      "rules": [
        {
          "name": "Password patterns",
          "type": "regex",
          "pattern": "(password|secret|key)\\s*=\\s*[\"'][^\"']{8,}[\"']",
          "file_pattern": "*.{go,js,py,java}",
          "message": "Potential hardcoded secret detected"
        }
      ],
      "outputs": [
        {
          "format": "markdown",
          "template": "text",
          "destination": "git:pr"
        }
      ]
    }
  ]
}
```

## Policy Management

### Disabling Policies
Temporarily disable a policy without removing it:

```json
{
  "vulnerability": [
    {
      "name": "Temporary disabled policy",
      "disabled": true,
      "actions": {
        "new": "block"
      }
    }
  ]
}
```

### Multiple Policies
Configure multiple policies for different scenarios:

```json
{
  "vulnerability": [
    {
      "name": "Critical vulnerabilities",
      "disabled": false,
      "actions": {"new": "block"},
      "rules": {"severity": [{"type": "eq", "value": "CRITICAL"}]}
    },
    {
      "name": "High vulnerabilities", 
      "disabled": false,
      "actions": {"new": "warn"},
      "rules": {"severity": [{"type": "eq", "value": "HIGH"}]}
    }
  ]
}
```

### Change-Specific Reporting
Configure different outputs for different change types:

```json
{
  "vulnerability": [
    {
      "name": "Security improvements tracker",
      "disabled": false,
      "actions": {
        "removed": "info"
      },
      "outputs": [
        {
          "format": "markdown",
          "template": "text", 
          "destination": "file:security-improvements.md",
          "changes": ["removed"],
          "title": "Security Fixes in This Release"
        }
      ]
    }
  ]
}
```

## Best Practices

### Policy Design
- **Start restrictive**: Begin with blocking critical issues, then adjust
- **Use meaningful names**: Make policy purposes clear
- **Group related rules**: Organize policies by severity or type
- **Document decisions**: Include comments explaining policy choices

### Change Type Strategy
- **new**: Most restrictive actions to prevent regression
- **existing**: Balanced approach to manage technical debt
- **removed**: Informational to celebrate improvements
- **changed**: Review-based to understand impact

### Output Configuration
- **PR comments**: For immediate developer feedback
- **Issue creation**: For tracking and follow-up
- **File outputs**: For reports and metrics
- **Log outputs**: For monitoring and alerting

### Testing Policies
Before deploying new policies:
1. Test with representative code samples
2. Validate with team on non-critical branches
3. Monitor initial results and adjust as needed
4. Document policy rationale for future reference
