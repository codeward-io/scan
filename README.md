# scan

### Create the following workflow file in you Github repository
> .github/workflow/codeward-io-scan.yaml
```
name: Codeward
on:
  workflow_dispatch:
  pull_request:
jobs:
  Codeward:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    permissions:
      pull-requests: write #needed to write comments in PR
      contents: read
      packages: read
      issues: write
    steps:
      - name: Scan
        uses: codeward-io/actions/scan@main
        with:
          event: ${{ github.event_name }}
          repository: ${{ github.repository }}
          current_branch: ${{ github.ref }}
```     