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

## Getting started with notify-slack

This action is intended to send messages to Slack to inform all internal stakeholders that something has happened in GitHub, and can be implemented as desired in any Action.

In your working repo (where you are implementing `notify-slack`), add the following Secrets in Actions:
- SLACK_BOT_TOKEN -> this can be generated from api.slack.com, or found in BitWarden
- SLACK_CHANNEL -> this points the API to a specific channel in Slack by it's Channel ID, e.g., `C0123TEST`

In your chosen Action, you can add a Slack message as a simple job, such as
```yml
jobs:
  notify-slack:
    uses: KomodoHQ/git-utils/.github/workflows/slack-notify.yml@main
    with:
      message: "Your message goes here including the ${{ github.repository }} if you wanted!"
    secrets:
      SLACK_CHANNEL: ${{ secrets.SLACK_CHANNEL }}
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

Alternatively you can add conditional messages, such as sending one message if an action is successful and a different message if the action fails, such as
```yml
  jobs:
    deploy:
      name: Run Staging Deploy
      runs-on: ubuntu-latest
+     outputs:
+       deployed: ${{ steps.set-output.outputs.success }}
      steps:
        ...normal steps go here...
+       - name: Set output success flag
+         id: set-output
+         run: |
+           if [ "${{ steps.deploy-step.outcome }}" == "success" ]; then
+             echo "success=true" >> $GITHUB_OUTPUT
+           else
+             echo "success=false" >> $GITHUB_OUTPUT
+           fi

+ notify-success:
+   name: Slack Success Notification
+   needs: deploy
+   if: ${{ needs.deploy.outputs.deployed == 'true' }}
+   uses: KomodoHQ/git-utils/.github/workflows/slack-notify.yml@main
+   with:
+     message: "Staging deploy succeeded for `${{ github.repository }}`"
+   secrets:
+     SLACK_CHANNEL: ${{ secrets.SLACK_CHANNEL }}
+     SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

+ notify-failure:
+   name: Slack Failure Notification
+   needs: deploy
+   if: ${{ needs.deploy.outputs.deployed == 'false' }}
+   uses: KomodoHQ/git-utils/.github/workflows/slack-notify.yml@main
+   with:
+     message: "Staging deploy failed for `${{ github.repository }}`"
+   secrets:
+     SLACK_CHANNEL: ${{ secrets.SLACK_CHANNEL }}
+     SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

```
