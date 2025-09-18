# Scanning Types

Codeward provides comprehensive security analysis through four distinct scanning types, each addressing specific aspects of application security and compliance.

## The Four Scanning Types

### 1. Vulnerability Scanning

**Purpose**: Detect security vulnerabilities in dependencies using Trivy scanner

**What it scans:**
- Known security vulnerabilities (CVEs) in dependencies
- Security issues in package versions
- Vulnerability status and fix availability

**Scanner**: [Trivy](https://trivy.dev/) - Industry-standard vulnerability scanner with embedded binaries

**Supported ecosystems:**
- Node.js (npm, yarn)
- Python (pip, poetry)
- Go (go modules)
- Java (Maven, Gradle)
- PHP (Composer)
- Ruby (Bundler)
- And many more

**Example configuration:**
```json
{
  "vulnerability": [
    {
      "name": "Critical vulnerabilities",
      "actions": {
        "new": "block",
        "existing": "warn"
      },
      "rules": {
        "severity": [
          {"type": "eq", "value": "CRITICAL"},
          {"type": "eq", "value": "HIGH"}
        ]
      }
    }
  ]
}
```

### 2. License Scanning

**Purpose**: Ensure legal compliance across all dependencies

**What it scans:**
- License types and categories
- License compatibility issues
- Unknown or missing licenses

**License classifications by risk level:**
- **Forbidden** (CRITICAL): Licenses that must be avoided
- **Restricted** (HIGH): Licenses requiring legal review  
- **Reciprocal** (MEDIUM): Licenses with source disclosure requirements
- **Notice/Permissive/Unencumbered** (LOW): Generally acceptable licenses
- **Unknown** (UNKNOWN): Unidentified or unclear licenses

**Example configuration:**
```json
{
  "license": [
    {
      "name": "License compliance",
      "actions": {
        "new": "warn"
      },
      "rules": {
        "severity": [
          {"type": "eq", "value": "CRITICAL"},
          {"type": "eq", "value": "HIGH"}
        ]
      }
    }
  ]
}
```

### 3. Package Scanning

**Purpose**: Track dependency changes between repository states

**What it monitors:**
- New packages added to project
- Package version changes
- Removed dependencies
- Dependency relationship changes (when dependency_tree is enabled)

**Change types tracked:**
- **New**: Packages introduced in the feature branch
- **Removed**: Packages removed from main in the feature branch
- **Changed**: Packages with version differences
- **Existing**: Packages present in both states

**Example configuration:**
```json
{
  "package": [
    {
      "name": "Dependency tracking",
      "actions": {
        "new": "info",
        "removed": "warn"
      },
      "rules": {
        "name": [
          {"type": "contains", "value": "security"}
        ]
      }
    }
  ]
}
```

### 4. Custom Validation

**Purpose**: Enforce project-specific security and compliance requirements

**Supported file types:**
- **text**: Plain text files, scripts, configuration files
- **json**: JSON configuration files, package manifests
- **yaml/yml**: YAML files, CI/CD configs, Kubernetes manifests
- **filesystem**: File and directory existence checks

**Common validation rules:**
- File content validation (contains, regex, etc.)
- Required file presence
- Configuration compliance
- Security requirement enforcement

**Example configuration:**
```json
{
  "validation": [
    {
      "name": "Dockerfile security",
      "path": "**/Dockerfile",
      "type": "text",
      "actions": {
        "new": "block"
      },
      "rules": [
        {
          "type": "contains",
          "value": "USER"
        }
      ]
    }
  ]
}
```

## How Scanning Types Work Together

### Comprehensive Security Coverage

Codeward performs all four scanning types in a coordinated manner:

1. **Trivy Scanner Execution**: Scans both main and feature branch repositories
2. **Result Processing**: Extracts vulnerabilities, licenses, and packages from Trivy output
3. **Diff Analysis**: Compares results between main and feature branches
4. **Custom Validation**: Runs file validation rules on repository content
5. **Policy Application**: Applies filtering rules and determines actions
6. **Report Generation**: Creates formatted outputs based on configuration

### Scanning Workflow

```
Repository Paths → Trivy Scanner → Raw Results → Diff Analysis → Policy Filtering → Report Generation
```

**Step-by-step process:**
1. Execute Trivy on main branch repository
2. Execute Trivy on feature branch repository (if different)
3. Extract vulnerability, license, and package data
4. Compare results to identify changes (new, existing, removed, changed)
5. Run custom validation policies on file content
6. Apply policy rules to filter results
7. Generate reports in specified formats
8. Send outputs to configured destinations

## Change Types and Actions

### Change Type Detection

Codeward identifies four types of changes between main and feature branches:

**NEW**: Items introduced in the feature branch
- New vulnerabilities from added dependencies
- New licenses from added packages
- New packages in dependency tree
- New validation failures

**EXISTING**: Items present in both branches
- Vulnerabilities present in both states
- Licenses unchanged between branches
- Packages with same versions
- Validation issues in both states

**REMOVED**: Items removed from main in feature branch
- Vulnerabilities fixed by removing/updating packages
- Licenses removed with package removal
- Packages removed from dependencies
- Validation issues resolved

**CHANGED**: Items modified between branches
- Vulnerabilities with different details
- License changes in updated packages
- Package version changes
- Validation rule changes

### Action Configuration

Each change type can have different actions:

```json
{
  "actions": {
    "new": "block",      // Fail CI/CD on new issues
    "existing": "warn",  // Log warnings for known issues
    "removed": "info",   // Celebrate improvements
    "changed": "warn"    // Review changes
  }
}
```

## Policy Integration

### Field-Specific Rules

Each scanning type supports specific rule fields:

**Vulnerability Rules:**
- `severity`: Filter by CRITICAL, HIGH, MEDIUM, LOW
- `pkgName`: Filter by package name patterns
- `vulnerabilityID`: Filter by CVE IDs
- `status`: Filter by vulnerability status

**License Rules:**
- `category`: Filter by license category
- `name`: Filter by license name patterns
- `severity`: Filter by license risk level

**Package Rules:**
- `name`: Filter by package name patterns
- `version`: Filter by version patterns
- `relationship`: Filter by dependency type

**Validation Rules:**
- File type specific validation rules
- Content-based validation (contains, regex, etc.)
- Existence checks for files and directories

### Output Customization

Each scanning type can generate customized outputs:

**Format Options:**
- `markdown`: GitHub-friendly format
- `html`: Web-displayable format
- `json`: API-consumable format

**Template Options:**
- `table`: Traditional tabular display
- `text`: Professional text format with emoji indicators

**Destination Options:**
- `log:stdout/stderr`: Console output
- `file:path`: Local file storage
- `git:pr`: GitHub PR comments
- `git:issue`: GitHub issue creation

## Best Practices

### Configuration Strategy

1. **Layered Security**: Use all four scanning types for comprehensive coverage
2. **Appropriate Actions**: Use `block` for critical security issues, `warn` for compliance
3. **Targeted Rules**: Focus rules on relevant security and compliance requirements
4. **Regular Updates**: Keep Trivy database updated for latest vulnerability information

### Performance Optimization

1. **Selective Scanning**: Disable unnecessary policies for faster execution
2. **Rule Optimization**: Use specific rules to reduce false positives
3. **Output Optimization**: Configure only needed output destinations
4. **Result Caching**: Use dependency_tree sparingly for large repositories

### Integration Patterns

1. **CI/CD Integration**: Focus on `new` changes to avoid blocking on existing issues
2. **Development Workflow**: Use `warn` actions during development, `block` for production
3. **Compliance Reporting**: Generate comprehensive reports for audit purposes
4. **Security Review**: Use detailed outputs for security team review

## Next Steps

- **[Diff Analysis](./diff-analysis.md)** - Understanding how Codeward compares repository states
- **[Policy System](./policy-system.md)** - Deep dive into policy configuration and behavior
- **[Configuration Architecture](./configuration-architecture.md)** - Understanding the configuration system


## 🚀 Performance and Efficiency

### **Optimized Scanning**
- **Incremental Analysis**: Only scan changes between branches
- **Cached Results**: Reuse scan data for unchanged dependencies
- **Parallel Processing**: Multiple scan types run concurrently
- **Smart Filtering**: Focus on relevant changes

### **Resource Management**
```json
{
  "global": {
    "ignore": {
      "paths": ["node_modules/", "vendor/", ".git/"],
      "file_extensions": [".log", ".tmp", ".cache"]
    }
  }
}
```

## 📊 Output Integration

All scanning types contribute to unified reporting:

### **Combined Reports**
```markdown
# Security Scan Results

## 🔒 Vulnerabilities (3 issues)
- 1 CRITICAL: Requires immediate attention
- 2 HIGH: Plan remediation within 7 days

## 📜 Licenses (1 issue)  
- 1 WARNING: GPL license review needed

## 📦 Packages (5 changes)
- 3 NEW: Added packages require review
- 2 UPDATED: Version changes detected

## ✅ Validations (2 issues)
- 1 FAILED: Dockerfile security check
- 1 PASSED: CI/CD configuration valid
```

### **Actionable Insights**
Each scan type provides specific, actionable recommendations:

- **Vulnerability**: Exact version to upgrade to
- **License**: Alternative packages with compatible licenses
- **Package**: Impact assessment of dependency changes
- **Validation**: Specific configuration fixes required

---

**Next Steps:**
- Learn about [Diff-Based Analysis](./diff-analysis.md)
- Understand the [Policy System](./policy-system.md)
- Explore [Configuration Architecture](./configuration-architecture.md)

## Related Topics

- [Diff-Based Analysis](./diff-analysis.md)
- [Policy System](./policy-system.md)
- [Vulnerability Policies](../policies/vulnerability.md)
- [License Policies](../policies/license.md)
- [Package Policies](../policies/package.md)
- [Custom Validation](../policies/validation.md)
