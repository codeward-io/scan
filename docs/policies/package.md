# Package Policies

Package policies monitor dependency changes and track package management activities, helping identify supply chain risks and maintain healthy dependency ecosystems.

## 🎯 Overview

Package policies track:

- **New package additions** to the dependency tree
- **Package version changes** and updates
- **Removed dependencies** and their impact
- **Package relationship changes** (direct vs indirect)

## 📋 Policy Structure

### **Basic Package Policy**

```json
{
  "package": [{
    "name": "Package change monitoring",
    "disabled": false,
    "actions": {
      "new": "info",
      "existing": "info",
      "removed": "warn",
      "changed": "info"
    },
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "fields": ["Name", "Version", "Relationship"],
      "changes": ["new", "removed", "changed"]
    }]
  }]
}
```

### **Policy Components**

#### **1. Name and Description**
```json
{
  "name": "Package change tracking",
  "description": "Monitors dependency changes and package management activities"
}
```

#### **2. Policy Disabling**
```json
{
  "disabled": false,  // Set to true to disable this policy
  "name": "Policy name"
}
```

Individual policies can be disabled by setting `"disabled": true. When disabled, the policy is skipped during scanning and the disabled policy names are logged for transparency.

#### **3. Actions Configuration**
```json
{
  "actions": {
    "new": "info",     // Track new packages
    "existing": "info", // Note existing packages
    "removed": "warn",  // Warn about removed packages
    "changed": "info"   // Track version changes
  }
}
```

## 🔍 Available Fields

### **Package-Specific Fields**

| Field | Description | Example Values |
|-------|-------------|----------------|
| `name` | Package name | `lodash`, `express`, `react` |
| `version` | Package version | `4.17.19`, `1.2.3`, `^2.0.0` |
| `relationship` | Dependency type | `direct`, `indirect` |
| `ecosystem` | Package ecosystem | `npm`, `pypi`, `maven`, `go` |

## 📊 Real-World Policy Examples

### **New Package Review Policy**

```json
{
  "package": [{
    "name": "New package review required",
    "disabled": false,
    "actions": {
      "new": "warn",
      "existing": "info",
      "removed": "info",
      "changed": "info"
    },
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "title": "📦 New Packages Added",
      "comment": "Please review these new dependencies for security and license compliance.",
      "fields": ["Name", "Version", "Relationship"],
      "changes": ["new"]
    }]
  }]
}
```

### **Dependency Removal Tracking**

```json
{
  "package": [{
    "name": "Dependency removal monitoring",
    "disabled": false,
    "actions": {
      "new": "info",
      "existing": "info",
      "removed": "block",
      "changed": "info"
    },
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "title": "🗑️ Dependencies Removed",
      "comment": "Dependency removal detected. Please verify this change is intentional.",
      "fields": ["Name", "Version", "Relationship"],
      "changes": ["removed"]
    }]
  }]
}
```

### **Version Change Monitoring**

```json
{
  "package": [{
    "name": "Version change tracking",
    "disabled": false,
    "actions": {
      "new": "info",
      "existing": "info",
      "removed": "info",
      "changed": "warn"
    },
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "title": "🔄 Package Version Changes",
      "fields": ["Name", "Version", "Relationship"],
      "changes": ["changed"]
    }]
  }]
}
```

### **Direct Dependency Focus**

```json
{
  "package": [{
    "name": "Direct dependency monitoring",
    "disabled": false,
    "actions": {
      "new": "warn",
      "existing": "info",
      "removed": "warn",
      "changed": "info"
    },
    "rules": {
      "relationship": [{"type": "eq", "value": "direct"}]
    },
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "title": "🎯 Direct Dependency Changes",
      "fields": ["Name", "Version", "Relationship"],
      "changes": ["new", "removed", "changed"]
    }]
  }]
}
```

## 📤 Output Customization

### **Field Selection**

**Essential Fields:**
```json
{
  "fields": ["Name", "Version", "Relationship"]
}
```

**Detailed Analysis:**
```json
{
  "fields": ["Name", "Version", "Relationship", "Ecosystem"]
}
```

### **Grouping Strategies**

**By Relationship:**
```json
{
  "group_by": ["Relationship"],
  "title": "Packages by Dependency Type"
}
```

**By Package Name:**
```json
{
  "group_by": ["Name"],
  "title": "Changes by Package"
}
```

## 📊 Package Management Best Practices

### **Dependency Types**

**Direct Dependencies:**
- Packages explicitly declared in your project
- Require careful review for security and licensing
- Should be regularly updated and audited

**Indirect/Transitive Dependencies:**
- Dependencies of your direct dependencies
- May introduce security vulnerabilities
- Updates typically managed through direct dependency updates

### **Change Types**

**New Packages:**
- Review for security vulnerabilities
- Check license compatibility
- Assess necessity and maintenance status

**Removed Packages:**
- Verify intentional removal
- Check for breaking changes in dependent code
- Update documentation and tests

**Version Changes:**
- Major version changes may require code updates
- Minor/patch updates generally safer
- Check changelog for breaking changes

---

**Next Steps:**
- Set up [Custom Validation](./validation.md)
- Learn about [Output Formats](../output/formats.md)
- Configure [Advanced Features](../configuration/overview.md)

## Related Topics

- [Custom Validation](./validation.md)
- [Output Formats](../output/formats.md)
- [Configuration Overview](../configuration/overview.md)
