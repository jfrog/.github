# PR Label Check

A GitHub Actions composite action that checks if a pull request has required labels from an allowed list.

## Usage

### Basic Usage

#### Option 1: Use with no inputs (all defaults)
```yaml
name: "PR Label Check"
on:
  pull_request:
    types: [opened, synchronize, reopened, labeled, unlabeled]

jobs:
  check-labels:
    name: Check PR Labels
    runs-on: ubuntu-latest
    steps:
      - name: Check if PR has allowed labels
        uses: ./.github/actions/pr-labels
```

#### Option 2: Use with custom allowed labels
```yaml
name: "PR Label Check"
on:
  pull_request:
    types: [opened, synchronize, reopened, labeled, unlabeled]

jobs:
  check-labels:
    name: Check PR Labels
    runs-on: ubuntu-latest
    steps:
      - name: Check if PR has allowed labels
        uses: ./.github/actions/pr-labels
        with:
          allowed-labels: 'new feature,bug,breaking change,ignore for release,improvement'
```

### Advanced Usage

```yaml
name: "PR Label Check"
on:
  pull_request:
    types: [opened, synchronize, reopened, labeled, unlabeled]

jobs:
  check-labels:
    name: Check PR Labels
    runs-on: ubuntu-latest
    steps:
      - name: Check if PR has allowed labels
        uses: ./.github/actions/pr-labels
        with:
          allowed-labels: 'enhancement,documentation,good first issue'
          require-labels: 'true'
          no-labels-message: 'Please add at least one label to this PR before merging.'
          invalid-labels-message: 'This PR must have one of these labels: {allowed}. Found: {current}'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `allowed-labels` | Comma-separated list of allowed labels | No | `'new feature,bug,breaking change,ignore for release,improvement'` |
| `require-labels` | Whether to require at least one label (true/false) | No | `'true'` |
| `no-labels-message` | Error message when no labels are present | No | `'This PR requires at least one label before it can be merged.'` |
| `invalid-labels-message` | Error message when labels are not in allowed list. Use `{allowed}` and `{current}` as placeholders | No | `'This PR must have at least one of these labels: {allowed}. Current labels: {current}'` |

## Outputs

| Output | Description |
|--------|-------------|
| `status` | Status of the check: `valid`, `no_labels` |
| `message` | Success message with current labels |
| `labels` | Comma-separated list of current PR labels |
| `error` | Error message (when check fails) |

## Examples

### Example 1: Basic label check
```yaml
- name: Check PR labels
  uses: ./.github/actions/pr-labels
  with:
    allowed-labels: 'bug,feature,enhancement'
```

### Example 2: Custom error messages
```yaml
- name: Check PR labels
  uses: ./.github/actions/pr-labels
  with:
    allowed-labels: 'bug,feature,enhancement'
    no-labels-message: '🚫 Please add a label to this PR'
    invalid-labels-message: '❌ Invalid label! Must be one of: {allowed}. Got: {current}'
```

### Example 3: Optional labels
```yaml
- name: Check PR labels
  uses: ./.github/actions/pr-labels
  with:
    allowed-labels: 'bug,feature,enhancement'
    require-labels: 'false'
```

## Default Behavior

When no inputs are provided, the action uses these defaults:
- **Allowed labels**: `'new feature'`, `'bug'`, `'breaking change'`, `'ignore for release'`, `'improvement'`
- **Require labels**: `true`
- **No labels message**: `'This PR requires at least one label before it can be merged.'`
- **Invalid labels message**: `'This PR must have at least one of these labels: {allowed}. Current labels: {current}'`

## How it works

1. The action retrieves the current PR labels using the GitHub API
2. It checks if the PR has any labels (if required)
3. It validates that at least one of the PR labels is in the allowed list
4. It outputs the results and fails the check if validation fails
5. If no inputs are provided, it gracefully falls back to default values

## Requirements

- The action runs on `ubuntu-latest`
- Requires GitHub CLI (`gh`) to be available (included in GitHub Actions runners)
- Must be triggered on pull request events 