# Output Formats

## Overview

Codeward supports three output formats: Markdown, HTML, and JSON. Each format can use different templates for presentation.

## Supported Formats

### Markdown
Markdown format for GitHub/GitLab integration:

**Table Template:**
```markdown
| VulnerabilityID | PkgName | Severity | FixedVersion |
|----------------|---------|----------|--------------|
| CVE-2023-12345 | lodash  | CRITICAL | 4.17.21     |
```

**Text Template:**
```markdown
🔴 CRITICAL: CVE-2023-12345 in lodash@4.17.19
   Fix: Update to lodash@4.17.21 or later
```

### HTML
HTML format for web viewing:

**Table Template:** Traditional HTML table format
**Text Template:** Styled HTML text format with rich formatting

### JSON
JSON format for programmatic processing:

```json
{
  "vulnerability": [
    {
      "VulnerabilityID": "CVE-2023-12345",
      "PkgName": "lodash",
      "Severity": "CRITICAL",
      "FixedVersion": "4.17.21"
    }
  ]
}
```

## Format Configuration

Configure output formats in policy outputs:

```json
{
  "outputs": [{
    "format": "markdown",
    "template": "table",
    "destination": "git:pr"
  }]
}
```

## Template Types

### Table Template
- Traditional table format (backward compatible)
- Best for tabular data, GitHub PR comments, issue reports

### Text Template
- Professional text-based format with improved readability
- Emoji indicators for severity levels
- Detailed vulnerability information
- Support for all policy types

## Format Combinations

- `format: "markdown", template: "table"` → Traditional markdown tables
- `format: "markdown", template: "text"` → Professional markdown text with emoji indicators
- `format: "html", template: "table"` → HTML tables for web display
- `format: "html", template: "text"` → Styled HTML text format with rich formatting
- `format: "json"` → JSON output (template setting ignored)
