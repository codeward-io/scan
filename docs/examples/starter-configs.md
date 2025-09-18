# Starter Configurations

Get up and running quickly with these proven Codeward configurations. Each configuration is designed for specific use cases and can be customized to meet your team's needs.

## Basic Security Configuration

Perfect for teams new to security scanning. Focuses on critical issues without overwhelming developers.

```json
{
  "vulnerability": [{
    "name": "Critical vulnerability protection",
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
    "outputs": [{
      "format": "markdown",
      "template": "table",
      "destination": "git:pr",
      "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
      "changes": ["new", "existing"]
    }]
  }],
  "license": [{
    "name": "License awareness",
    "disabled": false,
    "actions": {
      "new": "info",
      "existing": "info",
      "removed": "info",
      "changed": "info"
    },
    "outputs": [{
      "format": "markdown",
      "template": "table", 
      "destination": "git:pr",
      "fields": ["Name", "Category", "PkgName"],
      "changes": ["new"]
    }]
  }]
}
```

**What this does:**
- Blocks new critical vulnerabilities
- Warns about existing critical issues
- Tracks license information
- Provides clear PR feedback

## Comprehensive Security Configuration

More thorough security coverage for production environments.

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
      "outputs": [{
        "format": "markdown",
        "template": "table",
        "destination": "git:pr",
        "fields": ["VulnerabilityID", "PkgName", "Severity", "FixedVersion"],
        "changes": ["new", "existing"]
      }]
    },
    {
      "name": "High severity monitoring",
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
      "outputs": [{
        "format": "markdown",
        "template": "table",
        "destination": "git:pr", 
        "fields": ["VulnerabilityID", "PkgName", "InstalledVersion", "FixedVersion"],
        "changes": ["new"]
      }]
    }
  ],
  "license": [{
    "name": "License compliance",
    "disabled": false,
    "actions": {
      "new": "warn",
      "existing": "info",
      "removed": "info",
      "changed": "warn"
    },
    "rules": {
      "license": [{"type": "contains", "value": "GPL"}]
    },
    "outputs": [{
      "format": "markdown",
      "template": "text",
      "destination": "git:pr"
    }]
  }],
  "package": [{
    "name": "Package change tracking",
    "disabled": false,
    "actions": {
      "new": "info",
      "existing": "info",
      "removed": "info",
      "changed": "info"
    },
    "outputs": [{
      "format": "json",
      "destination": "file:package-changes.json"
    }]
  }]
}
```

**What this does:**
- Blocks critical vulnerabilities
- Warns about high severity issues
- Monitors license compliance
- Tracks all package changes

## Development Environment Configuration

Lightweight configuration for development and testing environments.

```json
{
  "vulnerability": [{
    "name": "Development security awareness",
    "disabled": false,
    "actions": {
      "new": "warn",
      "existing": "info",
      "removed": "info",
      "changed": "warn"
    },
    "rules": {
      "severity": [{"type": "eq", "value": "CRITICAL"}]
    },
    "outputs": [{
      "format": "json",
      "destination": "file:security-report.json"
    }]
  }],
  "package": [{
    "name": "Package change tracking",
    "disabled": false,
    "actions": {
      "new": "info",
      "existing": "info",
      "removed": "info",
      "changed": "info"
    },
    "outputs": [{
      "format": "markdown",
      "template": "text",
      "destination": "log:stdout"
    }]
  }]
}
```

**What this does:**
- Warns about critical vulnerabilities
- Tracks package changes
- Outputs JSON for tooling integration
- Lightweight for rapid development

## Custom Validation Configuration

Example configuration with custom validation rules.

```json
{
  "vulnerability": [{
    "name": "Security monitoring",
    "disabled": false,
    "actions": {
      "new": "warn",
      "existing": "info",
      "removed": "info",
      "changed": "warn"
    },
    "rules": {
      "severity": [
        {"type": "eq", "value": "CRITICAL"},
        {"type": "eq", "value": "HIGH"}
      ]
    },
    "outputs": [{
      "format": "markdown",
      "template": "table",
      "destination": "git:pr"
    }]
  }],
  "validation": [{
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
      },
      {
        "name": "API key patterns",
        "type": "regex", 
        "pattern": "api[_-]?key\\s*[:=]\\s*[\"'][a-zA-Z0-9]{20,}[\"']",
        "file_pattern": "*.{go,js,py,java,yaml,yml}",
        "message": "Potential API key detected"
      }
    ],
    "outputs": [{
      "format": "markdown",
      "template": "text",
      "destination": "git:pr"
    }]
  }]
}
```

**What this does:**
- Monitors security vulnerabilities
- Blocks hardcoded secrets
- Custom validation patterns
- Clear security feedback

## Configuration Comparison

| Configuration | Use Case | Blocking Policies | Performance | Learning Curve |
|---------------|----------|-------------------|-------------|----------------|
| **Basic Security** | New teams | Critical only | Fast | Low |
| **Comprehensive** | Production | Critical + License | Medium | Medium |
| **Development** | Local development | None | Very Fast | Very Low |
| **Custom Validation** | Security-focused | Critical + Secrets | Medium | Medium |

## Implementation Tips

### Start Small and Expand
1. **Week 1**: Use basic configuration with warnings only
2. **Week 2**: Enable blocking for critical vulnerabilities  
3. **Week 3**: Add license and package monitoring
4. **Week 4**: Implement custom validation rules

### Testing Your Configuration
1. Create a test branch with known vulnerabilities
2. Run Codeward with your configuration
3. Verify the expected policies trigger
4. Adjust actions (block/warn/info) as needed
5. Test with your team before deploying

### Configuration Management
- Version control your `config.json` with your project
- Document any custom rules or special requirements
- Review and update policies regularly

## Related Topics

- [Configuration Overview](../configuration/overview.md)
- [Policy System](../concepts/policy-system.md)
- [GitHub Actions Setup](../installation/github-actions.md)
- [Output Formats](../output/formats.md)
