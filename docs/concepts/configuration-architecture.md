# Configuration Architecture

## Overview

Codeward uses a simple but flexible configuration system that separates public and private settings (private settings should normally not be changed). This design allows teams to version control their scanning policies while keeping sensitive information secure.

## Configuration Structure

Codeward uses a JSON configuration files:
- **Configuration** (`config.json`): Public scanning policies and rules

This separation enables teams to:
- Version control their scanning policies in the main config
- Keep sensitive information out of version control
- Share consistent scanning rules across team members
- Customize local paths and credentials per environment

## Configuration File

The `config.json` file contains all public scanning policies and rules. This file should be version controlled with your project.

### Basic Structure
```json
{
  "vulnerability": [
    {
      "name": "Critical vulnerability policy",
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
  ],
  "license": [
    {
      "name": "License compliance check",
      "disabled": false,
      "actions": {
        "new": "warn",
        "existing": "info",
        "removed": "info",
        "changed": "info"
      },
      "rules": {
        "license": [{"type": "not_contains", "value": "GPL"}]
      },
      "outputs": [
        {
          "format": "markdown",
          "template": "text",
          "destination": "file:license-report.md"
        }
      ]
    }
  ],
  "package": [
    {
      "name": "Package tracking",
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
          "destination": "log:stdout"
        }
      ]
    }
  ],
  "validation": [
    {
      "name": "Custom validation rules",
      "disabled": false,
      "actions": {
        "new": "warn",
        "existing": "warn",
        "removed": "info",
        "changed": "warn"
      },
      "rules": [
        {
          "name": "No hardcoded secrets",
          "type": "regex",
          "pattern": "password|secret|key",
          "file_pattern": "*.go",
          "message": "Potential hardcoded secret detected"
        }
      ],
      "outputs": [
        {
          "format": "markdown",
          "template": "text",
          "destination": "git:issue"
        }
      ]
    }
  ]
}
```

### Policy Configuration Sections

#### Vulnerability Policies
Configure how security vulnerabilities are detected and reported:
- **Trivy-based scanning**: Uses industry-standard Trivy scanner
- **Severity filtering**: Target specific severity levels (CRITICAL, HIGH, MEDIUM, LOW)
- **Action mapping**: Different actions for new, existing, removed, and changed vulnerabilities

#### License Policies  
Track and validate software licenses:
- **License detection**: Identifies licenses in dependencies
- **Compliance rules**: Block or warn on specific license types
- **Change tracking**: Monitor license changes in dependencies

#### Package Policies
Monitor package and dependency changes:
- **Dependency tracking**: Track additions, removals, and updates
- **Package validation**: Custom rules for package management
- **Change analysis**: Compare package states between branches

#### Validation Policies
Custom validation rules for your codebase:
- **Regex patterns**: Search for specific patterns in code
- **File targeting**: Apply rules to specific file types
- **Custom messages**: Provide meaningful feedback to developers

### Configuration Fields

#### Required Paths
- **branch_path**: Path to the feature branch or pull request code
- **main_path**: Path to the main/master branch code for comparison
- **branch_results_path**: Where to store scan results for the branch
- **main_results_path**: Where to store scan results for the main branch

#### GitHub Integration (Optional)
- **token**: Personal access token for GitHub API access
- **owner**: GitHub organization or username
- **repo**: Repository name for creating issues and PR comments

### Environment-Specific Examples

#### CI/CD Environment
```json
{
  "branch_path": "/github/workspace",
  "main_path": "/tmp/main-checkout",
  "branch_results_path": "/tmp/branch-results.json",
  "main_results_path": "/tmp/main-results.json"
}
```

#### Local Development
```json
{
  "branch_path": "/Users/dev/myproject",
  "main_path": "/Users/dev/myproject-main",
  "branch_results_path": "/tmp/local-branch-results.json",
  "main_results_path": "/tmp/local-main-results.json"
}
```

## Configuration Usage Patterns

### Team Development
1. **Shared Policies**: Team agrees on scanning policies in `config.json`
2. **Version Control**: Main config is committed to repository
3. **Consistent Results**: Same policies applied across all environments

### CI/CD Integration
1. **Policy Enforcement**: Automated scanning with team-defined policies
2. **Automated Reporting**: Results posted to PRs and issues automatically
3. **Blocking Actions**: Critical findings can block deployments


## Best Practices

### Security
- **Use secrets or environment variables** for sensitive data in CI/CD
- **Rotate tokens** regularly for GitHub integration

### Maintainability  
- **Use descriptive names** for policies and rules
- **Group related policies** logically
- **Document custom validation rules** with clear messages
- **Test policy changes** before applying to main branch

### Performance
- **Disable unused policies** to speed up scanning
- **Target specific file patterns** for validation rules
- **Use appropriate output formats** for your workflow
- **Consider scan frequency** in automated environments
