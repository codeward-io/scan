# License Policies

License policies define how your system responds to license compliance issues detection to ensuring legal compliance and adherence to organizational license policies.

### **Equality Operators**
```json
{
  "severity": [
    {"type": "eq", "value": "CRITICAL"},    // Equals
    {"type": "ne", "value": "LOW"}          // Not equals
  ]
}
```

## 🎯 Overview

License policies use comprehensive license detection to identify:

- **License types** and categories (Forbidden, Restricted, Reciprocal, Notice, Permissive, Unencumbered, Unknown)
- **License risk levels** with severity ratings (CRITICAL, HIGH, MEDIUM, LOW, UNKNOWN)
- **Unknown or missing licenses** in packages
- **License violations** based on organizational policies

## 📋 Policy Structure

### **Basic License Policy**

```json
{
  "license": [{
    "name": "License compliance monitoring",
    "disabled": false,
    "actions": {
      "new": "warn",
      "existing": "info",
      "removed": "info",
      "changed": "info"
    },
    "rules": {
      "severity": [{"type": "eq", "value": "CRITICAL"}]
    },
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "fields": ["Name", "Category", "PkgName"],
      "changes": ["new", "existing"]
    }]
  }]
}
```

### **Policy Components**

#### **1. Name and Description**
```json
{
  "name": "License compliance policy",
  "description": "Ensures all dependencies comply with organizational license requirements"
}
```

#### **2. Policy Disabling**
```json
{
  "disabled": false,  // Set to true to disable this policy
  "name": "Policy name"
}
```

Individual policies can be disabled by setting `"disabled": true`. When disabled, the policy is skipped during scanning and the disabled policy names are logged for transparency.

#### **3. Actions Configuration**
```json
{
  "actions": {
    "new": "warn",      // Warn about new critical license issues
    "existing": "info", // Track existing license issues
    "removed": "info",  // Note license improvements
    "changed": "info"   // Track license changes
  }
}
```

#### **4. Filtering Rules**
```json
{
  "rules": {
    "severity": [
      {"type": "eq", "value": "CRITICAL"},
      {"type": "eq", "value": "HIGH"}
    ],
    "name": [
      {"type": "contains", "value": "GPL"},
      {"type": "ne", "value": "MIT"}
    ]
  }
}
```

## 🔍 Available Rule Fields

### **License-Specific Fields**

| Field | Description | Example Values |
|-------|-------------|----------------|
| `name` | License name | `MIT`, `Apache-2.0`, `GPL-3.0` |
| `severity` | License risk level | `CRITICAL`, `HIGH`, `MEDIUM`, `LOW`, `UNKNOWN` |
| `pkg_id` | Package identifier | `lodash@4.17.19`, `express@4.17.1` |
| `pkg_name` | Package name only | `lodash`, `express`, `react` |

## 🎛️ Rule Operators

### **Equality Operators**
```json
{
  "category": [
    {"type": "eq", "value": "copyleft"},    // Equals
    {"type": "ne", "value": "permissive"}   // Not equals
  ]
}
```

### **String Operators**
```json
{
  "name": [
    {"type": "contains", "value": "GPL"},      // Contains substring
    {"type": "not_contains", "value": "MIT"},  // Does not contain
    {"type": "hasPrefix", "value": "BSD"},     // Starts with
    {"type": "hasSuffix", "value": "-3.0"},    // Ends with
    {"type": "regex", "value": "^GPL-.*"}      // Regular expression
  ]
}
```

## 📊 Real-World Policy Examples

### **Permissive License Only Policy**

```json
{
  "license": [{
    "name": "Permissive license enforcement",
    "disabled": false,
    "actions": {
      "new": "block",
      "existing": "warn",
      "removed": "info"
    },
    "rules": {
      "severity": [
        {"type": "eq", "value": "CRITICAL"},
        {"type": "eq", "value": "HIGH"},
        {"type": "eq", "value": "UNKNOWN"}
      ]
    },
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "title": "🚫 Non-Permissive Licenses Detected",
      "fields": ["Name", "Category", "PkgName"],
      "changes": ["new", "existing"]
    }]
  }]
}
```

### **High Risk License Awareness**

```json
{
  "license": [{
    "name": "High risk license tracking",
    "disabled": false,
    "actions": {
      "new": "warn",
      "existing": "info",
      "removed": "info"
    },
    "rules": {
      "severity": [{"type": "eq", "value": "HIGH"}]
    },
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "title": "📜 High Risk License Notice",
      "comment": "These licenses require legal review and may have compliance requirements.",
      "fields": ["Name", "Category", "PkgName"],
      "changes": ["new"]
    }]
  }]
}
```

### **Unknown License Detection**

```json
{
  "license": [{
    "name": "Unknown license investigation",
    "disabled": false,
    "actions": {
      "new": "warn",
      "existing": "info",
      "removed": "info"
    },
    "rules": {
      "severity": [{"type": "eq", "value": "UNKNOWN"}]
    },
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "title": "❓ Unknown Licenses Require Review",
      "fields": ["Name", "Category", "PkgName"],
      "changes": ["new", "existing"]
    }]
  }]
}
```

## 📤 Output Customization

### **Field Selection**

**Essential Fields:**
```json
{
  "fields": ["Name", "Category", "PkgName"]
}
```

**Detailed Analysis:**
```json
{
  "fields": ["Name", "Category", "PkgID", "PkgName"]
}
```

### **Grouping Strategies**

**By License Category:**
```json
{
  "group_by": ["Category"],
  "title": "Licenses by Category"
}
```

**By Package:**
```json
{
  "group_by": ["PkgName"],
  "title": "Licenses by Package"
}
```

## 🚨 License Compliance Best Practices

### **License Categories**

**Permissive Licenses (Recommended):**
- MIT, Apache-2.0, BSD-2-Clause, BSD-3-Clause, ISC
- Allow commercial use, modification, and distribution
- Minimal compliance requirements

**Restricted/Reciprocal Licenses (Review Required):**
- GPL-2.0, GPL-3.0, AGPL-3.0, LGPL-2.1
- Require source code disclosure for derivatives
- May impact commercial distribution

**Proprietary Licenses (Avoid):**
- Commercial licenses with restrictions
- May require paid licensing or prohibit commercial use

### **Exception Management**

Exclude specific approved licenses:

```json
{
  "license": [{
    "name": "Standard license policy",
    "actions": {"new": "block", "existing": "warn"},
    "rules": {
      "severity": [{"type": "eq", "value": "HIGH"}],
      "name": [
        {"type": "ne", "value": "LGPL-2.1"},  // Approved exception
        {"type": "ne", "value": "BSD-2-Clause"}  // Approved permissive
      ]
    }
  }]
}
```

---

**Next Steps:**
- Configure [Package Policies](./package.md)
- Set up [Custom Validation](./validation.md)
- Learn about [Output Formats](../output/formats.md)

## Related Topics

- [Package Policies](./package.md)
- [Custom Validation](./validation.md)
- [Output Formats](../output/formats.md)
- [Configuration Overview](../configuration/overview.md)
