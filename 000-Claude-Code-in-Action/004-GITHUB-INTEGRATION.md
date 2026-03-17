# 004 — GitHub Integration on Claude Code

Step-by-step guide to integrate Claude Code with GitHub so you can mention `@claude` in issues and pull requests and have it act autonomously.

---

## Prerequisites

- A GitHub account with admin access to the target repository
- An Anthropic API key (from [console.anthropic.com](https://console.anthropic.com))

---

## Step 1 — Install the Claude GitHub App

1. Go to the Claude GitHub App page:
   [github.com/apps/claude](https://github.com/apps/claude)
2. Click **Install**.
3. Choose whether to install on all repositories or specific ones.
4. Click **Install & Authorize**.

---

## Step 2 — Add the Anthropic API Key to GitHub Secrets

1. In your repository, go to **Settings → Secrets and variables → Actions**.
2. Click **New repository secret**.
3. Set the name to `ANTHROPIC_API_KEY`.
4. Paste your Anthropic API key as the value.
5. Click **Add secret**.

---

## Step 3 — Create the GitHub Actions Workflow

Create the file `.github/workflows/claude.yml` in your repository:

```yaml
name: Claude Code

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  issues:
    types: [opened]
  pull_request_review:
    types: [submitted]

jobs:
  claude:
    if: |
      (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review' && contains(github.event.review.body, '@claude')) ||
      (github.event_name == 'issues' && contains(github.event.issue.body, '@claude'))
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      issues: write
    steps:
      - uses: anthropics/claude-code-action@beta
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

Commit and push this file to the default branch.

---

## Step 4 — Trigger Claude in a PR or Issue

Once the workflow is in place, mention `@claude` anywhere in:

- An **issue body** or comment
- A **pull request** comment or review

### Example prompts

```
@claude Can you review this PR and suggest improvements?
```

```
@claude Fix the failing tests in this PR.
```

```
@claude Implement the feature described in this issue.
```

Claude will respond as a GitHub Actions bot, post comments with its findings, and can push commits directly to the PR branch.

---

## Step 5 (Optional) — Allow Claude to Push Commits

By default Claude can read and comment. To let it push code to PR branches, ensure the workflow has the `contents: write` permission (already included above) and that the repository allows Actions to create and approve pull requests:

1. Go to **Settings → Actions → General**.
2. Under **Workflow permissions**, select **Read and write permissions**.
3. Check **Allow GitHub Actions to create and approve pull requests**.
4. Click **Save**.

---

## How It Works

```
User mentions @claude in issue/PR
        ↓
GitHub Actions workflow triggers
        ↓
claude-code-action runs Claude Code in a sandboxed runner
        ↓
Claude reads the repo, plans, and acts
        ↓
Claude posts comments and/or pushes commits back to the branch
```

---

## Useful Links

- Official docs: https://docs.anthropic.com/en/docs/claude-code/github-actions
- Claude Code Action repo: https://github.com/anthropics/claude-code-action
- Claude GitHub App: https://github.com/apps/claude
