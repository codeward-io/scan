---
sidebar_position: 1
---

# Welcome to Codeward

**Codeward** is a tool that helps you gain control over your codebase. This is done through comprehensive security scanning and reporting tool developed by codeward.io that performs vulnerability, license, package, and custom validation scanning of code repositories.

## 🎯 What is Codeward?

Codeward is a policy-driven security scanner that performs comprehensive analysis of your code repositories, detecting:

- **🔒 Security Vulnerabilities** - Using Trivy scanner for comprehensive vulnerability detection
- **📜 License Compliance Issues** - Ensuring legal compliance across dependencies
- **📦 Package Changes** - Tracking dependency modifications between repository states
- **✅ Custom Validations** - Enforcing project-specific requirements through flexible validation rules

## 🔧 How Codeward Works

Codeward performs **diff-based analysis** by comparing two repository states (typically main branch vs feature branch) to identify:

- **New vulnerabilities** introduced in your changes
- **Removed dependencies** and their impact
- **Changed packages** and version differences
- **Policy violations** based on your configuration

This focused approach allows security teams to concentrate on actual changes rather than reviewing entire codebases.

## ✨ Key Features

### 🔄 **Diff-Based Analysis**
- Compares main branch vs feature branch states
- Identifies exactly what changed and its security impact
- Focuses security reviews on actual changes

### 📋 **Policy-Driven Approach**
- Configurable rules for vulnerability, license, package, and validation policies
- Flexible actions: `info`, `warn`, or `block` for different change types
- Individual policies can be enabled or disabled as needed

### 🚀 **CI/CD Integration**
- Works with any CI/CD system supporting Docker or Go
- Can be integrated into GitHub workflows
- Automated reporting with multiple output destinations

### 📊 **Flexible Reporting**
- Multiple output formats: Markdown, HTML, JSON
- Template options: table format or professional text format
- Customizable fields, grouping, and filtering
- Combined reports across multiple policy types
- Professional templates for stakeholder communication

### CI/CD Integration

Codeward can be integrated into any CI/CD system. See our [installation guides](./installation/github-actions) or  detailed setup instructions.

## 📖 What's Next?

- **[Core Concepts](./concepts/scanning-types.md)** - Learn about scanning types and policy systems
- **[Configuration](./configuration/overview.md)** - Configure policies for your security requirements
- **[Policy System](./concepts/policy-system.md)** - Understand how policies work and how to customize them

## 🏢 Production Ready

Codeward scales from individual projects to enterprise environments:

- **Policy-driven governance** with flexible configuration
- **Multiple output destinations** for different stakeholders
- **Template customization** for consistent reporting
- **CI/CD integration** options for any development workflow

---

**Ready to secure your development workflow?** Start with our [Docker setup guide](./installation/docker.md), [Github Action setup guide](./installation/github-actions.md) or explore [core concepts](./concepts/scanning-types.md).
