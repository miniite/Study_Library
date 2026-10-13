---
type: "practice"
difficulty: "Intermediate"
---

# How would you send a Slack or email notification when a deployment workflow fails?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Reusable_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To send a Slack or email notification when a GitHub Actions workflow fails, we need to integrate our CI/CD pipeline with these external services. This process involves configuring webhooks and using specific actions available in the GitHub Actions ecosystem. Let's break down the steps from first principles.

Firstly, we need to understand that the goal is to trigger an event (either sending a Slack message or an email) when a particular step fails in our workflow. We will use GitHub Actions built-in actions for sending notifications and configure them properly.

#### Triage

The question asks how to send a Slack or email notification upon failure of a deployment workflow in GitHub Actions. The interviewer is probing your understanding of:

GitHub Actions Workflow Configuration: How to structure the YAML file.
Action Execution Context: Understanding when actions are executed and their states (success, failure).
External Service Integration: Configuring Slack or email services within GitHub Actions.

#### Assumptions
You have a basic understanding of GitHub repositories and workflows.
You know how to create and modify YAML files in your repository.
The workflow in question is already defined but has not included notification steps for failure events.

#### Solution

To achieve this, we need to follow these steps:

Set Up Slack or Email Action: Use a suitable action from the GitHub Marketplace.
Configure Webhook URLs: Obtain and use the webhook URL(s) provided by your Slack workspace or email provider.
Conditionally Trigger Notifications on Failure: Ensure notifications are only sent when specific conditions (like step failure) occur.
Step-by-Step Solution

Choose an Action

For Slack: Use actions like roboflow/slack or bcoe/github-action-slack.
For Email: Consider using the actions/email action.

Modify Your Workflow YAML File Here is a sample workflow file that includes both Slack and email notifications:

YAML
name: Deployment

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v2

    # Your deployment steps here (e.g., build, test)

    - name: Run Deployment Script
      run: |
        if ! ./deploy.sh; then
          echo "Deployment failed"
        fi

    - name: Send Slack Notification on Failure
      if: ${{ failure() }}
      uses: roboflow/slack@v2
      with:
        webhook-url: https://hooks.slack.com/services/<TEAM_ID>/<CHANNEL_ID>/<TOKEN>  # Replace with actual URL

    - name: Send Email Notification on Failure
      if: ${{ failure() }}
      uses: actions/email@v1
      env:
        EMAIL_RECIPIENT: example@example.com  # Replace with actual email address
      with:
        from_name: Deployment Team
        subject: Deployment Failed
        body: |
          The deployment failed at step "Run Deployment Script".

Explanation of Key Components

if: ${{ failure() }}: This condition ensures that the notification steps are only triggered if a step fails.
Webhook URLs and email addresses need to be correctly configured based on your environment.

Testing Your Workflow You can test this by intentionally failing one of the deployment steps (e.g., ./deploy.sh returning an error) and then checking Slack or your email for the notification.

#### Closing Summary

By using GitHub Actions, we can effectively integrate external services like Slack or email to get real-time notifications about our CI/CD pipeline. This approach ensures that we are promptly informed of any issues, allowing us to address them quickly.

#### Common Mistakes
and Follow-Ups
Ignoring Conditional Checks: Forgetting to use the if: ${{ failure() }} condition can lead to unnecessary notifications.
Not Handling Secrets Properly: Storing sensitive information like webhook URLs or email addresses directly in your workflow file is not secure. Use GitHub secrets for such values.
Using Default Actions: Sometimes, using default actions without customization might miss specific details; always tailor the actions to meet your needs.

**Code:**

```yaml
name: Deployment

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v2

    # Your deployment steps here (e.g., build, test)

    - name: Run Deployment Script
      run: |
        if ! ./deploy.sh; then
          echo "Deployment failed"
        fi

    - name: Send Slack Notification on Failure
      if: ${{ failure() }}
      uses: roboflow/slack@v2
      with:
        webhook-url: https://hooks.slack.com/services/<TEAM_ID>/<CHANNEL_ID>/<TOKEN>  # Replace with actual URL

    - name: Send Email Notification on Failure
      if: ${{ failure() }}
      uses: actions/email@v1
      env:
        EMAIL_RECIPIENT: example@example.com  # Replace with actual email address
      with:
        from_name: Deployment Team
        subject: Deployment Failed
        body: |
          The deployment failed at step "Run Deployment Script".
```

