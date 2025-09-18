# Policy Enforcement Strategies

Effective AI governance requires strategic policy enforcement that balances security with developer productivity. Codeward provides flexible enforcement mechanisms that adapt to your team's needs and maturity level.

## 🎯 Enforcement Philosophy

The goal is **security by default** without hindering innovation. Effective enforcement:

- **Prevents critical security issues** from reaching production
- **Educates developers** about security best practices
- **Scales with team growth** and changing requirements
- **Adapts to AI tool evolution** and new threat patterns

## 📋 Enforcement Levels

Codeward supports three enforcement levels for each policy:

### 🔵 **Info** - Awareness Building
- **Purpose**: Track and monitor without blocking
- **Use Cases**: New policies, metrics collection, team education
- **Developer Impact**: Visible in reports, no workflow disruption

```json
{
  "name": "Track dependency changes",
  "action": "info",
  "rules": {
    "package_name": [{"type": "contains", "value": "react"}]
  }
}
```

### 🟡 **Warn** - Guided Improvement  
- **Purpose**: Alert developers while allowing progression
- **Use Cases**: Medium-risk issues, policy transition periods
- **Developer Impact**: Prominent warnings, can proceed with acknowledgment

```json
{
  "name": "License compliance warnings", 
  "action": "warn",
  "rules": {
    "severity": [{"type": "eq", "value": "HIGH"}]
  }
}
```

### 🔴 **Block** - Critical Protection
- **Purpose**: Prevent dangerous code from merging
- **Use Cases**: Critical vulnerabilities, compliance violations
- **Developer Impact**: Blocks PR merge until resolved

```json
{
  "name": "Block critical vulnerabilities",
  "action": "block", 
  "rules": {
    "severity": [{"type": "eq", "value": "CRITICAL"}]
  }
}
```

## 🚀 Progressive Enforcement Strategy

### **Phase 1: Foundation (Weeks 1-4)**

**Objective**: Establish awareness without disruption

```json
{
  "vulnerability": [{
    "name": "Critical vulnerability tracking",
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
    }
  }],
  "license": [{
    "name": "License compliance awareness",
    "actions": {
      "new": "info",
      "existing": "info",
      "removed": "info"  
    }
  }]
}
```

**Activities:**
- Deploy Codeward with info/warn actions
- Generate baseline security metrics
- Train teams on scan results interpretation
- Gather feedback on policy effectiveness

### **Phase 2: Selective Enforcement (Weeks 5-8)**

**Objective**: Block critical issues while maintaining developer velocity

```json
{
  "vulnerability": [{
    "name": "Critical vulnerability blocking",
    "actions": {
      "new": "block",        // Block new critical issues
      "existing": "warn",    // Plan remediation for existing
      "removed": "info"      // Track improvements
    },
    "rules": {
      "severity": [{"type": "eq", "value": "CRITICAL"}]
    }
  }],
  "license": [{
    "name": "High severity license blocking", 
    "actions": {
      "new": "block",        // Prevent new violations
      "existing": "warn",    // Address existing issues
      "removed": "info"
    },
    "rules": {
      "severity": [{"type": "eq", "value": "HIGH"}]
    }
  }]
}
```

**Activities:**
- Implement blocking for critical security issues
- Establish remediation workflows for existing issues
- Monitor developer productivity impact
- Refine policies based on real-world usage

### **Phase 3: Comprehensive Governance (Weeks 9+)**

**Objective**: Full security coverage with optimized policies

```json
{
  "vulnerability": [{
    "name": "Comprehensive vulnerability management",
    "actions": {
      "new": "block",
      "existing": "warn", 
      "removed": "info"
    },
    "rules": {
      "severity": [
        {"type": "eq", "value": "CRITICAL"},
        {"type": "eq", "value": "HIGH"},
        {"type": "eq", "value": "MEDIUM"}  // Expanded coverage
      ]
    }
  }],
  "validation": [{
    "name": "Security configuration enforcement",
    "path": "Dockerfile",
    "type": "text",
    "action": "block",
    "rules": [{
      "type": "contains",
      "value": "USER"  // Require non-root containers
    }]
  }]
}
```

## 🎯 Context-Aware Enforcement

Different change types require different approaches:

### **New Issues** - Prevent Introduction
```json
{
  "actions": {
    "new": "block"  // Always block new critical issues
  }
}
```

### **Existing Issues** - Managed Remediation
```json
{
  "actions": {
    "existing": "warn"  // Track without blocking development
  }
}
```

### **Removed Issues** - Positive Reinforcement
```json
{
  "actions": {
    "removed": "info"  // Celebrate security improvements
  }
}
```

## 🏢 Organizational Enforcement Patterns

### **Startup/Small Team Pattern**
- **Focus**: High-impact, low-noise policies
- **Approach**: Gradual introduction with team buy-in
- **Priorities**: Critical vulnerabilities, major license issues

```json
{
  "vulnerability": [{
    "name": "Critical only",
    "action": "block",
    "rules": {
      "severity": [{"type": "eq", "value": "CRITICAL"}]
    }
  }]
}
```

### **Enterprise Pattern**
- **Focus**: Comprehensive coverage with audit trails
- **Approach**: Detailed policies with compliance reporting
- **Priorities**: Full vulnerability spectrum, license compliance, custom validations

```json
{
  "vulnerability": [
    {
      "name": "Critical vulnerabilities", 
      "action": "block",
      "rules": {"severity": [{"type": "eq", "value": "CRITICAL"}]}
    },
    {
      "name": "High/Medium vulnerabilities",
      "action": "warn", 
      "rules": {
        "severity": [
          {"type": "eq", "value": "HIGH"},
          {"type": "eq", "value": "MEDIUM"}
        ]
      }
    }
  ],
  "license": [{
    "name": "Enterprise license compliance",
    "action": "block",
    "rules": {
      "severity": [
        {"type": "eq", "value": "CRITICAL"},
        {"type": "eq", "value": "UNKNOWN"}
      ]
    }
  }]
}
```

### **Open Source Pattern**
- **Focus**: Community standards and transparency
- **Approach**: Flexible policies encouraging contribution
- **Priorities**: Known vulnerabilities, permissive license compliance

```json
{
  "license": [{
    "name": "Permissive licenses preferred",
    "action": "warn",
    "rules": {
      "category": [{"type": "ne", "value": "permissive"}]
    }
  }]
}
```

## 📊 Enforcement Metrics

Monitor enforcement effectiveness:

### **Security Metrics**
- **Critical Issues Blocked**: Count of prevented security issues
- **Remediation Time**: Average time to fix security problems
- **Policy Violation Rate**: Trends in security compliance

### **Developer Experience Metrics**  
- **False Positive Rate**: Incorrect security alerts
- **Policy Override Requests**: Policies that need refinement
- **Development Velocity Impact**: Build time and merge delays

### **Organizational Metrics**
- **Security Coverage**: Percentage of code under governance
- **Compliance Score**: Overall adherence to security policies
- **Risk Reduction**: Measurable decrease in security exposure

## 🔧 Fine-Tuning Enforcement

### **Common Adjustments**

**Reduce False Positives:**
```json
{
  "rules": {
    "package_name": [
      {"type": "ne", "value": "known-safe-package"}
    ],
    "severity": [
      {"type": "eq", "value": "CRITICAL"}
    ]
  }
}
```

**Add Contextual Rules:**
```json
{
  "rules": {
    "severity": [{"type": "eq", "value": "HIGH"}],
    "fixed_version": [{"type": "exists"}]  // Only if fix available
  }
}
```

**Environment-Specific Policies:**
```json
{
  "validation": [{
    "name": "Production configuration",
    "path": "prod.yml",
    "action": "block",
    "rules": [{"type": "contains", "value": "ssl: true"}]
  }]
}
```

## 🚨 Emergency Procedures

### **Policy Bypass for Urgent Issues**
```bash
# Temporary bypass using environment variable
export CODEWARD_BYPASS_CRITICAL="CVE-2023-12345"
```

### **Hotfix Workflow**
1. **Immediate Response**: Bypass blocking for critical hotfixes
2. **Security Review**: Manual security assessment
3. **Follow-up**: Address security issues in subsequent PR

### **Policy Rollback**
```json
{
  "name": "Temporary rollback",
  "action": "warn",  // Changed from "block"
  "comment": "Rolled back due to production issue - review scheduled"
}
```

## 🔄 Continuous Improvement

### **Regular Policy Reviews**
- **Monthly**: Review metrics and developer feedback
- **Quarterly**: Assess policy effectiveness and coverage
- **Annually**: Major policy updates and strategy alignment

### **Feedback Integration**
- Developer surveys on policy impact
- Security team assessment of coverage gaps  
- Management review of business impact

### **Adaptation to AI Evolution**
- Monitor new AI tool capabilities and risks
- Update policies for emerging threat patterns
- Integrate community best practices

---

**Next Steps:**
- Implement [Best Practices for AI Governance](./best-practices.md)
- Configure specific [Vulnerability Policies](../policies/vulnerability.md)
- Set up [Output and Reporting](../output/formats.md)

## Related Topics

- [Best Practices for AI Governance](./best-practices.md)
- [Vulnerability Policies](../policies/vulnerability.md)
- [License Policies](../policies/license.md)
- [Output Formats and Destinations](../output/formats.md)
