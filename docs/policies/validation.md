# Custom Validation Policies

Custom validation policies enforce project-specific security and compliance requirements by validating file contents, configurations, and project structure.

## 🎯 Overview

Custom validation policies support:

- **Text file validation** for configuration files, scripts, and documentation
- **JSON/YAML validation** for structured configuration files
- **File existence checks** for required project files
- **Content pattern matching** with flexible rules

## 📋 Policy Structure

### **Basic Validation Policy**

```json
{
  "validation": [{
    "name": "Dockerfile security validation",
    "disabled": false,
    "path": "Dockerfile",
    "type": "text",
    "action": "block",
    "rules": [{
      "type": "contains",
      "value": "USER"
    }],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "fields": ["key", "reason", "passing"]
    }]
  }]
}
```

### **Policy Components**

#### **1. Name and Description**
```json
{
  "name": "Dockerfile security validation",
  "description": "Ensures Dockerfiles follow security best practices"
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

#### **3. Target Configuration**
```json
{
  "path": "Dockerfile",    // File path pattern
  "type": "text"          // File type: text, json, yaml
}
```

#### **4. Action Configuration**
```json
{
  "action": "block"       // Action: block, warn, info
}
```

#### **5. Validation Rules**
```json
{
  "rules": [{
    "type": "contains",   // Rule type
    "key": "USER",        // Key to check (for JSON/YAML)
    "value": "root"       // Value to match
  }]
}
```

## 🔍 Supported File Types

### **Text Files**
Validate plain text files with pattern matching:

```json
{
  "validation": [{
    "name": "Dockerfile USER directive",
    "path": "Dockerfile",
    "type": "text",
    "action": "block",
    "rules": [{
      "type": "contains",
      "value": "USER"
    }]
  }]
}
```

### **JSON Files**
Validate JSON files with key-value checks:

```json
{
  "validation": [{
    "name": "package.json security",
    "path": "package.json",
    "type": "json",
    "action": "warn",
    "rules": [{
      "type": "exists",
      "key": "scripts.test"
    }]
  }]
}
```

### **YAML Files**
Validate YAML files with structured checks:

```json
{
  "validation": [{
    "name": "CI/CD configuration",
    "path": ".github/workflows/*.yml",
    "type": "yaml",
    "action": "block",
    "rules": [{
      "type": "eq",
      "key": "jobs.build.security",
      "value": "true"
    }]
  }]
}
```

## 🎛️ Rule Types

### **Text Validation Rules**

| Rule Type | Description | Example |
|-----------|-------------|---------|
| `contains` | Text must contain substring | `{"type": "contains", "value": "USER"}` |
| `not_contains` | Text must not contain substring | `{"type": "not_contains", "value": "password"}` |
| `regex` | Text must match regex pattern | `{"type": "regex", "value": "^USER \\w+"}` |

### **JSON/YAML Validation Rules**

| Rule Type | Description | Example |
|-----------|-------------|---------|
| `exists` | Key must exist | `{"type": "exists", "key": "version"}` |
| `not_exists` | Key must not exist | `{"type": "not_exists", "key": "debug"}` |
| `eq` | Value must equal | `{"type": "eq", "key": "env", "value": "production"}` |
| `ne` | Value must not equal | `{"type": "ne", "key": "log_level", "value": "debug"}` |
| `contains` | Value must contain substring | `{"type": "contains", "key": "name", "value": "test"}` |

## 📊 Real-World Policy Examples

### **Dockerfile Security**

```json
{
  "validation": [
    {
      "name": "Non-root user required",
      "path": "Dockerfile",
      "type": "text",
      "action": "block",
      "rules": [{
        "type": "contains",
        "value": "USER"
      }],
      "outputs": [{
        "format": "markdown",
        "destination": "git:pr",
        "title": "🐳 Dockerfile Security Issues",
        "fields": ["key", "reason", "passing"]
      }]
    },
    {
      "name": "No privileged containers",
      "path": "Dockerfile",
      "type": "text",
      "action": "block",
      "rules": [{
        "type": "not_contains",
        "value": "--privileged"
      }]
    }
  ]
}
```

### **CI/CD Security**

```json
{
  "validation": [{
    "name": "GitHub Actions security",
    "path": ".github/workflows/*.yml",
    "type": "yaml",
    "action": "warn",
    "rules": [
      {
        "type": "not_contains",
        "key": "jobs.*.steps[*].run",
        "value": "curl | sh"
      },
      {
        "type": "eq",
        "key": "permissions.contents",
        "value": "read"
      }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "title": "🔧 CI/CD Security Validation",
      "fields": ["key", "reason", "passing"]
    }]
  }]
}
```

### **Package Configuration**

```json
{
  "validation": [{
    "name": "package.json validation",
    "path": "package.json",
    "type": "json",
    "action": "warn",
    "rules": [
      {
        "type": "exists",
        "key": "version"
      },
      {
        "type": "exists",
        "key": "license"
      },
      {
        "type": "not_contains",
        "key": "scripts.postinstall",
        "value": "curl"
      }
    ],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "title": "📦 Package Configuration Issues",
      "fields": ["key", "reason", "passing"]
    }]
  }]
}
```

### **Required Files Check**

```json
{
  "validation": [{
    "name": "Required files exist",
    "path": "README.md",
    "type": "text",
    "action": "info",
    "rules": [{
      "type": "exists"
    }],
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "title": "📄 Missing Required Files",
      "fields": ["key", "reason", "passing"]
    }]
  }]
}
```

## 📤 Output Customization

### **Field Selection**

**Standard Fields:**
```json
{
  "fields": ["key", "reason", "passing"]
}
```

### **Grouping Strategies**

**By File Type:**
```json
{
  "group_by": ["path"],
  "title": "Validation Issues by File"
}
```

## 🚨 Validation Best Practices

### **Security-First Approach**

**Dockerfile Validations:**
- Always require non-root USER
- Prevent privileged containers
- Avoid hardcoded secrets
- Use trusted base images

**CI/CD Validations:**
- Prevent shell script injection
- Require minimal permissions
- Validate workflow triggers
- Check for unsafe actions

**Configuration Validations:**
- Require license declarations
- Prevent debug settings in production
- Validate security configurations
- Check for deprecated features

### **Pattern Matching**

**Safe Patterns:**
```json
{
  "rules": [{
    "type": "regex",
    "value": "^USER \\w+$"  // Specific USER format
  }]
}
```

**Dangerous Patterns to Avoid:**
```json
{
  "rules": [{
    "type": "not_contains",
    "value": "password"  // Generic secrets
  }]
}
```

---

**Next Steps:**
- Learn about [Output Formats](../output/formats.md)
- Configure [Advanced Features](../configuration/overview.md)
- Explore [Real-World Examples](../examples/starter-configs.md)

## Related Topics

- [Output Formats](../output/formats.md)
- [Configuration Overview](../configuration/overview.md)
- [Starter Configurations](../examples/starter-configs.md)
