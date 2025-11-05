# AI README Antispam

A GitHub Action to detect spammy README edits using AI to protect your repository from spam pull requests.

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-AI%20README%20Antispam-blue.svg?colorA=24292e&colorB=0366d6&style=flat&longCache=true&logo=github)](https://github.com/marketplace/actions/ai-readme-antispam)

[![GitHub Repository](https://img.shields.io/badge/GitHub-Repository-181717.svg?style=flat&logo=github)](https://github.com/rbadillap/ai-readme-antispam)



https://github.com/user-attachments/assets/60b582f1-3cad-4e5d-8f27-7abacbf4d1bb



## Features

- **AI-Powered Detection**: Uses advanced AI models to analyze README changes
- **Smart Classification**: Categorizes changes as spam, unknown, or legitimate
- **Flexible Integration**: Use outputs to comment, label, or fail workflows
- **Fast & Reliable**: Built on Vercel AI Action infrastructure

## How It Works

1. **Detects README changes**: Extracts README file modifications from the current PR
2. **Analyzes with AI**: Uses AI to understand the context and intent of changes
3. **Returns results**: Provides structured outputs for you to act upon

The action identifies spam as:
- Promotional or irrelevant link additions
- Trivial changes without real value
- SEO link farming attempts

It recognizes legitimate changes as:
- Typo fixes
- Documentation improvements
- Technical examples and guides

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `api-key` | Yes | - | API key for AI Gateway (get one at [vercel.com/ai-gateway](https://vercel.com/ai-gateway)) |
| `github-token` | No | `${{ github.token }}` | [GITHUB_TOKEN](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) (issues: write, pull-requests: write) or a repo scoped PAT |
| `model` | No | `openai/gpt-4o` | AI model to use ([see supported models](https://vercel.com/ai-gateway/models)) |

## Outputs

| Output | Description |
|--------|-------------|
| `spam-type` | Type of spam detected: `spam`, `unknown`, or `none` |
| `analysis-reason` | Detailed reason for the classification |
| `raw-json` | Full JSON response from AI analysis |

## Usage

### Basic Example

```yaml
name: Spam Detection
on:
  pull_request:
    paths:
      - '**/README*'

jobs:
  detect-spam:
    runs-on: ubuntu-latest
    steps:
      - uses: rbadillap/ai-readme-antispam@v2
        id: spam-check
        with:
          api-key: ${{ secrets.AI_GATEWAY_API_KEY }}
          
      - run: |
          echo "Type: ${{ steps.spam-check.outputs.spam-type }}"
          echo "Reason: ${{ steps.spam-check.outputs.analysis-reason }}"
```

### Advanced Examples

#### Auto-comment on spam detection

```yaml
- uses: rbadillap/ai-readme-antispam@v2
  id: spam-check
  with:
    api-key: ${{ secrets.AI_GATEWAY_API_KEY }}

- uses: actions/github-script@v7
  if: steps.spam-check.outputs.spam-type == 'spam'
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    script: |
      github.rest.issues.createComment({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        body: "The changes to the README in this PR may not align with the project documentation standards. Please review and ensure the modifications add meaningful value to the project.\n\n" +
              "Reason: ${{ steps.spam-check.outputs.analysis-reason }}\n\n" +
              "Feel free to update the PR if needed!"
      })
```

#### Auto-close PR on spam detection

```yaml
- uses: rbadillap/ai-readme-antispam@v2
  id: spam-check
  with:
    api-key: ${{ secrets.AI_GATEWAY_API_KEY }}

- uses: actions/github-script@v7
  if: steps.spam-check.outputs.spam-type == 'spam'
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    script: |
      await github.rest.issues.createComment({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        body: "This PR may have been opened accidentally or contains changes that don't align with our documentation guidelines. I'm closing it now, but feel free to open a new PR with more substantial documentation improvements!\n\n" +
              "Reason: ${{ steps.spam-check.outputs.analysis-reason }}"
      })
      
      await github.rest.pulls.update({
        owner: context.repo.owner,
        repo: context.repo.repo,
        pull_number: context.issue.number,
        state: 'closed'
      })
```

#### Fail workflow on spam

```yaml
- uses: rbadillap/ai-readme-antispam@v2
  id: spam-check
  with:
    api-key: ${{ secrets.AI_GATEWAY_API_KEY }}

- name: Fail if spam detected
  if: steps.spam-check.outputs.spam-type == 'spam'
  run: |
    echo "::error::README changes may not meet documentation standards: ${{ steps.spam-check.outputs.analysis-reason }}"
    exit 1
```

#### Label PRs automatically

```yaml
- uses: rbadillap/ai-readme-antispam@v2
  id: spam-check
  with:
    api-key: ${{ secrets.AI_GATEWAY_API_KEY }}

- uses: actions/github-script@v7
  if: steps.spam-check.outputs.spam-type == 'spam'
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    script: |
      github.rest.issues.addLabels({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        labels: ['spam', 'needs-review']
      })
```

## Live Example

See this action in action: [rbadillap/test-readme-antispam](https://github.com/rbadillap/test-readme-antispam) - A live repository using this action to automatically detect and close spam PRs.

## License

MIT © [Ronny Badilla](https://github.com/rbadillap)

## Support

- [Documentation](https://github.com/rbadillap/ai-readme-antispam)
- [Report issues](https://github.com/rbadillap/ai-readme-antispam/issues)
- [Request features](https://github.com/rbadillap/ai-readme-antispam/issues/new)
