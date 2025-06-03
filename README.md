# git-utils

A central repository of utility scripts and GitHub Actions to enhance our development workflows, automate routine tasks, and integrate GitHub with third-party tools such as Slack and Jira.

## Purpose

This repo serves as a shared toolkit to:
- Automate common Dev and QA-related tasks.
- Standardise and reuse GitHub Actions across multiple projects.
- Improve communication through Slack notifications.
- Integrate GitHub workflows with Jira for better traceability.

## Features

- **Slack Notifications**  
  Trigger alerts in Slack when workflows start, succeed, or fail.

- **Jira Automation**  
  Automatically create or update Jira tickets from GitHub Actions based on repository events.

- **Reusable Actions**  
  Define and publish modular GitHub Actions for use across the organisation’s repositories.

- **CI/CD Enhancements**  
  Add consistency and automation to common CI/CD workflows.

---

## Getting Started with `slack-notify`

The `slack-notify.yml` reusable workflow sends rich-formatted deployment messages to a specified Slack channel. It uses [Slack Block Kit](https://api.slack.com/block-kit) to include structured fields and a direct link to the GitHub workflow run.

### Required Secrets

Set these in your repository's **Actions > Secrets and variables**:

| Secret Name       | Description                          |
|-------------------|--------------------------------------|
| `SLACK_BOT_TOKEN` | Slack Bot Token (found in Bitwarden) |
| `SLACK_CHANNEL`   | Slack Channel ID, e.g. `C0123ABCDEF` |

---

### Basic Usage

Add the following job to any workflow:

```yml
jobs:
  notify:
    uses: KomodoHQ/git-utils/.github/workflows/slack-notify.yml@main
    with:
      status: "succeeded"
      repository: ${{ github.repository }}
      branch: ${{ github.ref_name }}
      commit_sha: ${{ github.sha }}
      actor: ${{ github.actor }}
      action_name: ${{ github.workflow }}
    secrets:
      SLACK_CHANNEL: ${{ secrets.SLACK_CHANNEL }}
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

This will post a message that includes the repo name, branch, commit, author, and a button to view the workflow run.

### Conditional Notifications

To send a success or failure message based on the result of a deployment:

```yml
jobs:
  deploy:
    name: Run Staging Deploy
    runs-on: ubuntu-latest
    outputs:
      deployed: ${{ steps.set-output.outputs.success }}
    steps:
      - name: Your deploy logic
        id: deploy-step
        run: |
          echo "doing something"
          sleep 1

      - name: Set output success flag
        id: set-output
        run: |
          if [ "${{ steps.deploy-step.outcome }}" == "success" ]; then
            echo "success=true" >> $GITHUB_OUTPUT
          else
            echo "success=false" >> $GITHUB_OUTPUT
          fi

  notify-success:
    name: Slack Success Notification
    needs: deploy
    if: ${{ needs.deploy.outputs.deployed == 'true' }}
    uses: KomodoHQ/git-utils/.github/workflows/slack-notify.yml@main
    with:
      status: "succeeded"
      repository: ${{ github.repository }}
      branch: ${{ github.ref_name }}
      commit_sha: ${{ github.sha }}
      actor: ${{ github.actor }}
      action_name: ${{ github.workflow }}
    secrets:
      SLACK_CHANNEL: ${{ secrets.SLACK_CHANNEL }}
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

  notify-failure:
    name: Slack Failure Notification
    needs: deploy
    if: ${{ needs.deploy.outputs.deployed == 'false' }}
    uses: KomodoHQ/git-utils/.github/workflows/slack-notify.yml@main
    with:
      status: "failed"
      repository: ${{ github.repository }}
      branch: ${{ github.ref_name }}
      commit_sha: ${{ github.sha }}
      actor: ${{ github.actor }}
      action_name: ${{ github.workflow }}
    secrets:
      SLACK_CHANNEL: ${{ secrets.SLACK_CHANNEL }}
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```
