# Review Droid - Automated Code Review for GitHub Pull Requests

An intelligent, automated code review system powered by **Factory AI**. Review Droid analyzes code diffs for critical issues and submits precise, line-anchored comments directly to your Pull Request.

## How to Use

The Review Droid action is designed to run automatically whenever a Pull Request is opened or updated.

Add the following workflow file to your repository at `.github/workflows/review-droid.yml`.

### 1\. Create Secrets (Setup)

This action requires one secret to be stored in your repository or organization:

| Secret Name | Description | Required |
| :--- | :--- | :--- |
| **`FACTORY_API_KEY`** | Your Factory AI API key to run Droid CLI. | Yes |

### 2\. Workflow File (`.github/workflows/review-droid.yml`)

```yaml
name: Automated Review Droid

# Trigger on PR events, skipping draft PRs
on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]

# Cancel any previous runs for the same PR to save time
concurrency:
  group: review-droid-${{ github.event.pull_request.number }}
  cancel-in-progress: true

jobs:
  code-review:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    # Skip automated code review for draft PRs
    if: github.event.pull_request.draft == false
    
    # Grant necessary permissions for the default GITHUB_TOKEN to write reviews
    permissions:
      pull-requests: write
      contents: read
      issues: write

    steps:
      - name: Run Factory AI Review Droid
        # Replace <OWNER>/<REPO> with your actual organization/repository name and version tag
        uses: <OWNER>/<REPO>@v1 
        with:
          # --- REQUIRED INPUTS ---
          # Note: The secret name should match the one created in step 1
          factory-api-key: ${{ secrets.FACTORY_API_KEY }} 
          
          # Pass the specific PR context variables required by the action
          pr-number: ${{ github.event.pull_request.number }} 
          pr-head-sha: ${{ github.event.pull_request.head.sha }}
```

-----

## Inputs

| Name | Description | Required |
| :--- | :--- | :--- |
| `factory-api-key` | The API key for the Factory AI service. | **Yes** |
| `pr-number` | The Pull Request number being reviewed (`${{ github.event.pull_request.number }}`). | **Yes** |
| `pr-head-sha` | The commit SHA of the PR head branch for correct checkout (`${{ github.event.pull_request.head.sha }}`). | **Yes** |
