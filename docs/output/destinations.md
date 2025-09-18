# Output Destinations

## Overview

Codeward supports four output destinations for sending scan results and reports.

## Supported Destinations

### File Output
Write results to local files:

```json
{
  "outputs": [{
    "format": "markdown",
    "destination": "file:./results/scan-report.md"
  }]
}
```

### Console Output
Output results to stdout or stderr:

```json
{
  "outputs": [{
    "format": "markdown",
    "destination": "log:stdout"
  }]
}
```

### GitHub PR Comments
Post results as comments on GitHub pull requests:

```json
{
  "outputs": [{
    "format": "markdown",
    "destination": "git:pr"
  }]
}
```

### GitHub Issues
Create GitHub issues for security findings:

```json
{
  "outputs": [{
    "format": "markdown",
    "destination": "git:issue"
  }]
}
```

## Destination Configuration

Configure destinations in policy outputs:

```json
{
  "outputs": [{
    "format": "markdown",
    "template": "table",
    "destination": "git:pr",
    "title": "Security Scan Results",
    "comment": "Review the following security findings"
  }]
}
```

## Combined Reports

Multiple policies can be combined into single outputs by destination:

```json
{
  "vulnerability": [{
    "name": "Vuln policy",
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "title": "Combined Security Report"
    }]
  }],
  "license": [{
    "name": "License policy", 
    "outputs": [{
      "format": "markdown",
      "destination": "git:pr",
      "title": "Combined Security Report"
    }]
  }]
}
```

Policies with the same destination and title are combined into a single report.

## Environment Variables for Git Integration

- `GITHUB_TOKEN`: GitHub API token for PR comments and issue creation
- `GITHUB_OWNER`: Repository owner/organization
- `GITHUB_REPO`: Repository name  
- `GITHUB_PR_NR`: Pull request number
