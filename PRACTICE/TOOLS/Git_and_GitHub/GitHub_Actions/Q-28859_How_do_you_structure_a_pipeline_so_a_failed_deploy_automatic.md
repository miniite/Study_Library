---
type: "practice"
difficulty: "Intermediate"
---

# How do you structure a pipeline so a failed deploy automatically opens an incident ticket and pages on-call?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Reusable_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting up a GitHub Actions workflow to automate deployments, it’s crucial to ensure that any failure during the deployment process triggers both an incident ticket and pages the on-call team. This requires structuring the pipeline with specific steps that handle errors properly and integrate with external services like Slack or PagerDuty.

#### Triage

The question is essentially about creating a robust GitHub Actions workflow that includes error handling to open an incident ticket and notify the on-call team in case of a deployment failure. We need to define how to catch errors, interface with external APIs for ticket creation and paged notifications, and ensure these steps are executed conditionally based on the outcome of the previous steps.

#### Solution
Define the Workflow Structure: First, we'll create a GitHub Actions workflow file (.yml) that outlines the deployment process.
Error Handling: Use error handling to catch any failure during the deployment and trigger subsequent actions.
Incident Ticket Creation: Integrate with an incident management tool like Jira or GitHub Issues via API calls to open a new ticket.
Paging On-Call Team: Use a paging service (e.g., PagerDuty) to notify the on-call team.
Step-by-Step Solution

Create the Workflow File:

Define the workflow with necessary steps for different stages of deployment.
Add conditional logic to detect failures and trigger appropriate actions.

Error Handling in GitHub Actions:

YAML
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      # Checkout code, run tests, etc.
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Run Tests
        run: make test  # or equivalent command
        if: failure()
        continue-on-error: true

      - name: Deploy to Staging
        run: make deploy-staging
        env:
          API_KEY: ${{ secrets.API_KEY }}
        if: always()
        continue-on-error: true

Error Handling and Ticket Creation:

YAML
jobs:
  # Continue from the previous steps...

  create-incident-ticket:
    runs-on: ubuntu-latest
    needs: deploy  # Ensure this job only runs if deployment fails.
    steps:
      - name: Create Incident Ticket
        run: |
          curl -X POST https://api.jira.example.com/issue \
              --header "Content-Type: application/json" \
              --data '{"fields": {"project": {"key": "PROJ"}, "summary": "Deployment Failed", "description": "Failed during deploy-staging"}}'
        if: failure()

Paging the On-Call Team:

YAML
jobs:
  # Continue from the previous steps...

  page-oncall:
    runs-on: ubuntu-latest
    needs: create-incident-ticket  # Ensure this job only runs if an incident ticket is created.
    steps:
      - name: Page On-Call Team
        run: |
          curl -X POST https://api.pagerduty.com/incidents \
              --header "Content-Type: application/json" \
              --header "Authorization: Token token=${{ secrets.PAGERDUTY_API_KEY }}" \
              --data '{"incident": {"type": "incident", "title": "Deployment Failed"}}'
        if: always()

#### Closing Summary

By structuring the GitHub Actions workflow to include specific steps for error handling, ticket creation, and paged notifications, we ensure that any deployment failure is promptly addressed. The use of conditional logic (if: failure()) ensures that only necessary actions are triggered based on the current state.

#### Common Mistakes/Follow-Ups
Ignoring Conditional Logic: Failing to conditionally handle errors can lead to unnecessary or missed steps.
API Key Exposure: Always securely manage API keys and secrets within GitHub Secrets.
Paging vs. Incident Ticketing: Ensure that paged notifications are used as a fallback for immediate attention, while incident tickets capture the detailed failure information.

**Code:**

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Run Tests
        run: make test
        if: failure()
        continue-on-error: true

      - name: Deploy to Staging
        run: make deploy-staging
        env:
          API_KEY: ${{ secrets.API_KEY }}
        if: always()
        continue-on-error: true

  create-incident-ticket:
    runs-on: ubuntu-latest
    needs: deploy
    steps:
      - name: Create Incident Ticket
        run: |
          curl -X POST https://api.jira.example.com/issue \
              --header "Content-Type: application/json" \
              --data '{"fields": {"project": {"key": "PROJ"}, "summary": "Deployment Failed", "description": "Failed during deploy-staging"}}'
        if: failure()

  page-oncall:
    runs-on: ubuntu-latest
    needs: create-incident-ticket
    steps:
      - name: Page On-Call Team
        run: |
          curl -X POST https://api.pagerduty.com/incidents \
              --header "Content-Type: application/json" \
              --header "Authorization: Token token=${{ secrets.PAGERDUTY_API_KEY }}" \
              --data '{"incident": {"type": "incident", "title": "Deployment Failed"}}'
        if: always()
```

**Example input / output:**

```
Example input to the workflow

Checkout Code: Output: Successfully checked out code.

Run Tests: Output: All tests passed.

Deploy to Staging: Output: Deployment successful in staging environment.

CODE
Copy

- After a failure during deployment:

```yaml
- Deploy to Staging (failure):
  Output: Deployment failed with error "Error deploying to staging".

- Create Incident Ticket:
  Output: Incident ticket created successfully in Jira with summary "Deployment Failed".

- Page On-Call Team:
  Output: Notification sent to on-call team via PagerDuty.
```

**Flow:**

1. Yes
2. No
3. Checkout Code
4. Run Tests
5. Test Success?
6. Deploy to Staging
7. Create Incident Ticket
8. Page On-Call Team

