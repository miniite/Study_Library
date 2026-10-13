---
type: "practice"
difficulty: "Intermediate"
---

# How do you make a deployment job require manual approval before running in production?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ GitLab CI Fundamentals](<../../../../TOOLS/GitLab/GitLab_CI/GitLab_CI_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to ensure that any deploy job triggered on our GitLab CI pipeline does not run automatically and requires a manual approval before proceeding to the production environment. This is crucial for maintaining control over deployments and preventing accidental rollouts.

#### Triaging the Question

The interviewer is probing how you handle manual approvals in your CI/CD pipelines, specifically using GitLab CI. You need to understand that this involves configuring GitLab CI to include a step where an admin or another authorized user can manually approve the deployment before it proceeds.

#### First-Principles Solution
Define the Deployment Job: In your .gitlab-ci.yml file, define a job for deploying to production.
Add Manual Approval Step: Use GitLab’s rules:changes feature to enforce that this job can only run if manually triggered.
Use Variables and Secrets: Optionally, leverage GitLab’s secret variables or environment-specific tokens to control access.

Here's the step-by-step approach:

Define the Deployment Job: In your .gitlab-ci.yml, you might have a section like this for deploying to production:

YAML
deploy_production:
  stage: deploy
  script:
    - echo "Deploying to Production..."
    - # Your deployment steps here
  only:
    refs:
      - main

Add Manual Approval: To require manual approval, you can use the rules section in GitLab CI:

YAML
deploy_production:
  stage: deploy
  script:
    - echo "Deploying to Production..."
    - # Your deployment steps here
  only:
    refs:
      - main
  rules:
    - if: '$CI_COMMIT_REF_NAME == "main"'
      when: manual  # This ensures the job runs manually and requires approval

Environment-Specific Tokens (Optional): To add an extra layer of security, you can use environment-specific tokens stored as secrets in GitLab:

YAML
deploy_production:
  stage: deploy
  script:
    - echo "Deploying to Production..."
    - # Your deployment steps here
  only:
    refs:
      - main
  variables:
    DEPLOY_TOKEN: $DEPLOY_PRODUCTION_TOKEN  # This token is set as a GitLab Secret
  rules:
    - if: '$CI_COMMIT_REF_NAME == "main" && $DEPLOY_TOKEN == "$DEPLOY_PRODUCTION_TOKEN"'
      when: manual

Review and Test: After configuring your CI job, review the .gitlab-ci.yml file to ensure that the rules:manual directive is correctly applied. You can test by manually triggering a pipeline from GitLab's UI or using the API.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
Common Mistake: Failing to include the when: manual clause in the rules, which would result in automatic execution.

#### Follow-Up
Question: You might be asked about additional security measures, such as integrating with external authentication systems (e.g., Okta) or implementing multi-factor authentication.

By following these steps, you ensure that any deployment to production is done only after explicit manual approval, maintaining high standards of operational discipline and minimizing the risk of accidental rollouts.

**Code:**

```yaml
# .gitlab-ci.yml example

stages:
  - deploy

deploy_production:
  stage: deploy
  script:
    - echo "Deploying to Production..."
    - # Your deployment steps here
  only:
    refs:
      - main
  rules:
    - if: '$CI_COMMIT_REF_NAME == "main"'
      when: manual
```

