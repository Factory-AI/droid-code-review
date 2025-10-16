# Factory Droid Review

An AI-powered code review GitHub Action using Factory's Droid to analyze code changes and identify critical issues. This action automatically reviews pull requests, focusing on bugs, security vulnerabilities, and critical code problems.

## Features

- **AI-Powered Analysis**: Uses Factory's Droid CLI to analyze code changes and identify critical issues
- **Full Repository Context**: Checks out the PR head branch so Droid can analyze any relevant file in the repository (not just the diff) before leaving review comments
- **Inline Code Comments**: Posts targeted review comments directly on specific lines with optional fix suggestions
- **Duplicate Prevention**: Checks existing comments to avoid redundant feedback
- **Focused Review Scope**: Prioritizes critical bugs and security issues over style concerns
- **Debug Artifacts**: Uploads diagnostic logs on failure for troubleshooting

## Quick Start

Add this to your repository's `.github/workflows/review-droid.yml`:

```yaml
name: Droid Review

permissions:
  pull-requests: write  # Needed for leaving PR comments
  contents: read
  issues: write

on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]

# Cancel previous runs for the same PR
concurrency:
  group: review-droid-${{ github.event.pull_request.number }}
  cancel-in-progress: true

jobs:
  code-review:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    # Skip draft PRs
    if: github.event.pull_request.draft == false

    steps:
      - uses: Factory-AI/droid-code-review@latest
        with:
          factory-api-key: ${{ secrets.FACTORY_API_KEY }}
          pr-number: ${{ github.event.pull_request.number }}
          pr-head-sha: ${{ github.event.pull_request.head.sha }}
```

## Configuration Options

### Action Inputs

| Input | Description | Default | Required |
|-------|-------------|---------|----------|
| `factory-api-key` | The API key to run Droid Exec which powers the Review Droid. | None | Yes |
| `pr-number` | The Pull Request number being reviewed | None | Yes |
| `pr-head-sha` | The commit SHA of the PR head branch for correct checkout | None | Yes |

## Code Review Capabilities

### Issues Actively Detected

- **Dead/Unreachable Code**: Code after return/throw/break, if(false) blocks
- **Control Flow Bugs**: Missing break statements, switch fallthrough issues
- **Async/Await Errors**: Missing await, unhandled promise rejections, incorrect promise handling
- **React-Specific Issues**: State mutations, useEffect dependency problems
- **Operator Mistakes**: Wrong equality operators (== vs ===), assignment in conditions
- **Array/Loop Errors**: Off-by-one errors, incorrect indexing
- **Type Coercion Issues**: Problematic type conversions affecting behavior
- **Null/Undefined Errors**: Potential null dereferences, missing checks
- **Resource Management**: Unclosed files/connections, memory leaks
- **Security Vulnerabilities**: SQL/XSS injection risks, unvalidated environment variables
- **Concurrency Problems**: Race conditions, synchronization issues
- **Error Handling**: Missing error handling for critical operations
- **Recursion Issues**: Missing base cases, stack overflow risks
- **Regex Problems**: Catastrophic backtracking vulnerabilities

### What's NOT Reviewed

The action intentionally skips:
- Code style, formatting, or naming conventions
- Minor performance optimizations
- Architectural decisions or design patterns
- Feature completeness or functionality (unless broken)
- Test coverage (unless tests are clearly broken)

## Installation & Setup

### GitHub Actions

Follow the Quick Start guide above. The action will:
1. Install Factory's Droid CLI using the official installer
2. Configure the CLI with your API key
3. Execute the code review analysis

### Creating the Secret

1. Go to your repository **Settings** → **Secrets and variables** → **Actions**
2. Create a new repository secret named `FACTORY_API_KEY`
3. Paste your Factory API key (get one at [factory.ai](https://app.factory.ai/settings/api-keys))

## Technical Implementation

### How It Works

1. **Checkout**: Action checks out the repository at the PR head commit with full history
2. **CLI Installation**: Downloads and installs Droid CLI from https://app.factory.ai/cli
3. **PR Analysis**: Fetches complete PR data from GitHub URL (diff, comments, metadata)
4. **Code Review**: Runs `droid exec --skip-permissions-unsafe` with the review prompt
5. **Comment Submission**: Posts inline comments via GitHub REST API using curl
6. **Error Handling**: Uploads debug artifacts if the review fails

### Review Limits

- Maximum of 10 inline comments per review (prioritizing most critical issues)
- Comments only on modified lines in the PR
- Suggestions provided only when fixes are certain
- No PR approval/rejection to avoid blocking merge workflows

### Debug Information

On failure, the action uploads:
- The review prompt used
- Droid execution logs
- Console output logs

These artifacts are retained for 7 days to help diagnose issues.

## Support

For issues or questions:
- Open an issue in this repository
- Visit [Factory Documentation](https://docs.factory.ai) for API details

## License

MIT License - see [LICENSE](LICENSE) file for details
