# Diff-Based Analysis

Codeward's core functionality is **diff-based analysis** - comparing the security state between two versions of your repository to identify exactly what changed. This approach provides focused, actionable security feedback by highlighting only the differences.

## How Diff Analysis Works

### Comparison Process

Codeward compares two repository states:
1. **Main branch state**: The baseline for comparison (usually main/master)
2. **Feature branch state**: The changes being evaluated (usually a PR branch)

The tool scans both states and compares the results to identify:
- **New issues**: Problems introduced in the feature branch
- **Removed issues**: Problems fixed in the feature branch  
- **Existing issues**: Problems present in both versions
- **Changed issues**: Problems with modified details (severity, etc.)

## Change Classification

Codeward categorizes differences into four types of changes:

### New Changes
**Definition**: Issues introduced in the feature branch that weren't present in main

**Example**: Adding a dependency with a known vulnerability
```json
{
  "change_type": "new",
  "vulnerability_id": "CVE-2023-12345",
  "package": "express@4.17.1",
  "severity": "CRITICAL"
}
```

**Typical Policy**: Configure with `"new": "block"` to prevent new security issues

### Existing Changes  
**Definition**: Issues present in both main and feature branches

**Example**: Known vulnerability that exists in both versions
```json
{
  "change_type": "existing", 
  "vulnerability_id": "CVE-2023-67890",
  "package": "lodash@4.17.19",
  "severity": "HIGH"
}
```

**Typical Policy**: Configure with `"existing": "warn"` or `"existing": "info"` to track without blocking

### Removed Changes
**Definition**: Issues that were resolved in the feature branch

**Example**: Vulnerability fixed by updating a dependency
```json
{
  "change_type": "removed",
  "vulnerability_id": "CVE-2023-11111", 
  "package": "old-crypto@1.0.0",
  "severity": "CRITICAL"
}
```

**Typical Policy**: Configure with `"removed": "info"` to acknowledge improvements

### Changed Changes
**Definition**: Issues with modified details between branches

**Example**: Vulnerability with updated severity rating
```json
{
  "change_type": "changed",
  "vulnerability_id": "CVE-2023-22222",
  "package": "image-processor@2.1.0", 
  "severity": "HIGH",
  "previous_severity": "MEDIUM"
}
```

**Typical Policy**: Configure with `"changed": "warn"` to review modifications

## Policy Configuration for Changes

### Action Configuration
Configure different actions for each change type in your policies:

```json
{
  "vulnerability": [{
    "name": "Critical vulnerability policy",
    "disabled": false,
    "actions": {
      "new": "block",      // Block new critical issues
      "existing": "warn",  // Warn about existing issues  
      "removed": "info",   // Log resolved issues
      "changed": "warn"    // Review changed issues
    },
    "rules": {
      "severity": [{"type": "eq", "value": "CRITICAL"}]
    },
    "outputs": [
      {
        "format": "markdown",
        "template": "table", 
        "destination": "git:pr",
        "fields": ["VulnerabilityID", "PkgName", "Severity"],
        "changes": ["new", "existing"]
      }
    ]
  }]
}
```

### Change-Specific Outputs
Control which changes appear in different outputs:

```json
{
  "vulnerability": [{
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
        "changes": ["removed"],  // Only show resolved issues
        "title": "Security Improvements This Sprint"
      }
    ]
  }]
}
```

## Practical Examples

### Pull Request Workflow
When a developer creates a pull request:

1. **CI scans both branches**: Main branch and PR branch are scanned
2. **Changes identified**: Codeward compares the scan results
3. **Policies applied**: Each change type triggers configured actions
4. **Results reported**: Focused report shows only the differences

### Example Text Output
```markdown
## [codeward.io] Vulnerability Report

**📋 Policy:** Vulnerability issues

### 📁 `go/go.mod`
#### ⚠️ **EXISTING VULNERABILITIES**
##### 🟡 **Medium: CVE-2024-24786**
- **Vulnerability ID:** CVE-2024-24786
- **Package ID:** `google.golang.org/protobuf@v1.30.0`
- **Package:** `google.golang.org/protobuf`
- **Installed Version:** v1.30.0
- **Fix Available:** Update to `1.33.0` or later
- **Status:** fixed
- **Severity:** MEDIUM
- **Relationship:** indirect
- **Parents:** golang-test-dependencies 
- **Targets:** go/go.mod 
##### 🟠 **High: CVE-2023-39325**
- **Vulnerability ID:** CVE-2023-39325
- **Package ID:** `golang.org/x/net@v0.9.0`
- **Package:** `golang.org/x/net`
- **Installed Version:** v0.9.0
- **Fix Available:** Update to `0.17.0` or later
- **Status:** fixed
- **Severity:** HIGH
- **Relationship:** direct
- **Parents:** golang-test-dependencies 
- **Targets:** go/go.mod 
```

### Example Table Output
```markdown
## [codeward.io] Vulnerability Report

> Policy: **Vulnerability issues**
#### `go/go.mod`

Detection: **existing** - Action: **warn**

| VulnerabilityID | PkgID | PkgName | InstalledVersion | FixedVersion | Status | Severity | Relationship | Children | Parents | Targets |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| CVE-2020-28483 | github.com/gin-gonic/gin@v1.6.3 | github.com/gin-gonic/gin | v1.6.3 | 1.7.7 | fixed | HIGH | indirect | N/A | golang-test-dependencies | go/go.mod |
| CVE-2023-26125 | github.com/gin-gonic/gin@v1.6.3 | github.com/gin-gonic/gin | v1.6.3 | 1.9.0 | fixed | MEDIUM | indirect | N/A | golang-test-dependencies | go/go.mod |
| CVE-2023-29401 | github.com/gin-gonic/gin@v1.6.3 | github.com/gin-gonic/gin | v1.6.3 | 1.9.1 | fixed | MEDIUM | indirect | N/A | golang-test-dependencies | go/go.mod |
```

## Benefits of Diff Analysis

### Focused Feedback
- **Relevant results**: Only shows what changed in your PR
- **Actionable items**: Clear about what needs attention
- **Reduced noise**: Existing issues don't overwhelm new ones

### Clear Responsibility
- **Attribution**: Links security changes to specific code changes
- **Accountability**: Developers see impact of their dependency choices
- **Education**: Learn about security implications of changes

### Continuous Improvement
- **Progress tracking**: Monitor security improvements over time
- **Positive reinforcement**: Celebrate when vulnerabilities are fixed
- **Trend analysis**: Understand security trajectory of your project
