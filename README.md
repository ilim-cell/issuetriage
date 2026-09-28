# Issue Fix Status Action

A GitHub Action that bridges the gap between PR merges, production releases, and reporter confirmations. Track fixes from merge to release without prematurely closing issues.

---

## Setup

Add the minimal workflow file below to `.github/workflows/fix-status.yml`. All inputs are pre-configured with safe defaults.

```yaml
name: Issue Fix Status

on:
  pull_request_target:
    types: [opened, edited, closed]
  workflow_run:
    workflows: ['Release'] # Name of your production release workflow
    types: [completed]
  schedule:
    - cron: '0 7 * * *' # Daily cleanup check
  workflow_dispatch:

jobs:
  fix-status:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      issues: write
      pull-requests: write
    steps:
      - uses: your-username/issue-fix-status-action@v1

```

---

## Arguments and Inputs

All inputs are required by default in the action specification, but automatically fall back to sensible defaults if omitted from your workflow.

| Input | Description | Default |
| --- | --- | --- |
| `github-token` | GitHub access token (`github.token` or PAT). | `${{ github.token }}` |
| `keyword-action` | Strategy for closing keywords (`warn`, `reopen`, `reopen except maintainer`, `ignore`). | `'warn'` |
| `pending-label` | Label applied when a PR merges to the main branch. | `'fixed (pending release)'` |
| `pending-label-color` | Color hex code for the pending label. | `'0e8a16'` |
| `confirm-label` | Label applied when a release is published. | `'please confirm close'` |
| `confirm-label-color` | Color hex code for the confirm label. | `'048c23'` |
| `half-label` | Label for issues where a fix is only partial. | `'half'` |
| `skip-labels` | JSON string array of labels to ignore. | `'["summary"]'` |
| `confirm-days` | Number of days to wait for reporter reply before closing. | `'14'` |
| `ref-regex` | Regex pattern to detect referenced issues in PRs. | Standard issue ref regex |
| `closing-keyword-regex` | Regex pattern to detect GitHub closing keywords (`fixes #123`). | Standard closing keyword regex |

---

## Keyword Action Strategies

When contributors use native GitHub keywords (e.g., `fixes #123`), GitHub normally closes the issue immediately upon PR merge. Select one of four strategies to handle this behavior:

* **`warn`** *(Default)*: Comments on the PR suggesting plain reference syntax (e.g., `Refs #123`). Reacting with a 👍 on the warning comment authorizes the action to reopen the issue on merge.
* **`reopen`**: Automatically reopens any issue closed by a PR merge keyword so it can proceed through the verification pipeline.
* **`reopen except maintainer`**: Reopens issues closed by keywords *unless* the PR author is a repo maintainer (`OWNER`, `MEMBER`, or `COLLABORATOR`).
* **`ignore`**: Disables keyword checks and permits GitHub's default issue-closing behavior.

---

## Configuration Examples

```yaml
name: Issue Fix Status

on:
  pull_request_target:
    types: [opened, edited, closed]
  workflow_run:
    workflows: ['Release']
    types: [completed]
  schedule:
    - cron: '0 7 * * *'
  workflow_dispatch:

jobs:
  fix-status:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      issues: write
      pull-requests: write
    steps:
      - uses: your-username/issue-fix-status-action@v1
        with:
          keyword-action: 'reopen except maintainer'
          pending-label: 'status: pending release'
          confirm-label: 'status: needs verification'
          confirm-days: '7'

```

```yaml
name: Issue Fix Status

on:
  pull_request_target:
    types: [opened, edited, closed]
  workflow_run:
    workflows: ['Release']
    types: [completed]
  schedule:
    - cron: '0 7 * * *'
  workflow_dispatch:

jobs:
  fix-status:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      issues: write
      pull-requests: write
    steps:
      - uses: your-username/issue-fix-status-action@v1
        with:
          template-pending-comment: |
            🚀 **Fix merged!** Landed in PR #{PR_NUMBER}.
            This fix will be included in our next release.
          template-release-comment: |
            🎉 **Released in {VERSION}!**
            @{LOGIN}, please let us know if this resolves your issue.

```

**Central Workflow (`org-name/.github/.github/workflows/shared-fix-status.yml`):**

```yaml
name: Shared Fix Status

on:
  workflow_call:

jobs:
  fix-status:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      issues: write
      pull-requests: write
    steps:
      - uses: your-username/issue-fix-status-action@v1

```

**Repository Caller Workflow (`.github/workflows/fix-status.yml`):**

```yaml
name: Issue Fix Status

on:
  pull_request_target:
    types: [opened, edited, closed]
  workflow_run:
    workflows: ['Release']
    types: [completed]
  schedule:
    - cron: '0 7 * * *'

jobs:
  fix-status:
    uses: org-name/.github/.github/workflows/shared-fix-status.yml@main

```
