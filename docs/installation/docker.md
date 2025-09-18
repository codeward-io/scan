# Docker Installation

Run Codeward in Docker containers for consistent, isolated environments.

## Quick Start

```bash
docker run --rm \
  -v /path/to/repository:/main \
  codeward-io/scan:latest
```

Codeward will scan the repository using the `config.json` in it or using the default configuration if it does not exist.

## Advanced Setup

```bash
docker run --rm \
  -v /path/to/main:/main \
  -v /path/to/branch:/branch \
  -v /path/to/config.json:/config.json \
  -v /path/to/private_config.json:/config_private.json \
  -e CONFIG_PATH=/config.json \
  -e PRIVATE_CONFIG_PATH=/config_private.json \
  codeward-io/scan:latest
```

## Environment Variables

**Configuration Loading:**
- `CONFIG_PATH`: Path to main configuration file
- `PRIVATE_CONFIG_PATH`: Path to private configuration file

**GitHub Integration:** (required for `git:pr` and `git:issue` destinations)
- `GITHUB_TOKEN`: GitHub API token
- `GITHUB_OWNER`: Repository owner
- `GITHUB_REPO`: Repository name
- `GITHUB_PR_NR`: Pull request number
- `CI_EVENT`: Event type (`pr` or `main`)

If no private configuration is provided, Codeward uses embedded defaults based on the environment.

## Private Configuration Structure (Advanced)

### Basic Structure

```json
{
  "main_path": "/path/to/main/branch",
  "branch_path": "/path/to/feature/branch",
  "main_results_path": "/results/main.json",
  "branch_results_path": "/results/branch.json",
  "diff_results_path": "/results/diff.json",
  "diff_results_filtered_path": "/results/diff_filtered.json",
  "enable_results": false
}
```

### Configuration Fields

**Repository Paths:**
- `main_path` (string): Path to main/base branch repository
- `branch_path` (string): Path to feature branch repository (optional for single-branch scans)

**Result Paths:**
- `main_results_path` (string): Where to store main branch scan results
- `branch_results_path` (string): Where to store feature branch scan results
- `diff_results_path` (string): Where to store diff analysis results
- `diff_results_filtered_path` (string): Where to store filtered diff results

**Control Options:**
- `enable_results` (boolean): Whether to save intermediate result files

## Next Steps
- **[GitHub Actions](./github-actions.md)** - CI/CD integration
- **[Configuration Guide](../configuration/overview.md)** - Setup policies
