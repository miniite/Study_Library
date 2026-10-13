---
type: "practice"
difficulty: "Deep"
---

# How do you implement a smoke test after deploy that automatically rolls back if health checks fail?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ GitLab CI Fundamentals](<../../../../TOOLS/GitLab/GitLab_CI/GitLab_CI_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

Smoke testing is crucial to ensure that newly deployed applications are functioning as expected before they fully go live. In this scenario, we will implement a GitLab CI pipeline that includes a smoke test after deployment and automatically rolls back the application if any health checks fail.

#### Triage

The core of this question revolves around automating post-deployment verification through GitLab CI/CD pipelines. The interviewer is probing your understanding of:

GitLab CI Pipeline Structure: How to set up stages, jobs, and triggers.
Health Checks: Implementing and interpreting health check results.
Rollback Mechanism: Automating the rollback process if smoke tests fail.

#### Assumptions
The application is deployed via GitLab CI/CD pipeline.
Health checks can be executed using a specific command or endpoint that returns a status code or message.
Rollbacks are initiated through a GitLab API or by rolling back the environment's deployment configuration.
Step-by-Step Solution

Define the Pipeline Structure

A typical GitLab CI/CD pipeline consists of stages, each containing jobs. We'll add a smoke-tests stage after the deploy stage.

Implement Health Checks

Define a job that performs smoke tests. This job can use curl to send HTTP requests and verify expected responses.

Configure Rollback Mechanism

If any health check fails, this failure should trigger a rollback mechanism using GitLab API or environment configuration changes.
1. Define the Pipeline Structure

Here’s an example of how you might structure your pipeline YAML file:

YAML
stages:
  - deploy
  - smoke-tests

deploy-job:
  stage: deploy
  script:
    - echo "Deploying application..."
    - # Your deployment commands here (e.g., `docker-compose up`)

smoke-test-job:
  stage: smoke-tests
  script:
    - echo "Performing smoke tests..."
    - curl --fail http://localhost/health-check || { echo "Health check failed"; exit 1; }
2. Implement Health Checks

The health check job will perform a simple HTTP request to the /health-check endpoint of your application. If the response status code is not 200, it will fail, and the pipeline will stop.

3. Configure Rollback Mechanism

If any smoke test fails, we need to configure GitLab CI/CD to automatically roll back the deployment. This can be done by setting up a job that uses the GitLab API or environment configuration changes.

YAML
rollback-job:
  stage: rollback
  script:
    - echo "Rolling back application..."
    - # Use GitLab API or environment configuration tools to revert deployment

#### Closing Summary

By implementing this pipeline, you ensure that your application is thoroughly tested after deployment. If any health checks fail, the automatic rollback job will be triggered to revert changes and prevent issues from going live.

#### Common Mistakes/Follow-Ups
Not Defining Health Checks: Failing to define clear health check criteria can lead to false negatives or positives.
Manual Rollback: Relying on manual intervention instead of automated rollback mechanisms can introduce delays.
Inadequate Error Handling: Not properly handling errors in the smoke test job can cause pipeline failures.

#### Example
Worked Input->Output
PLAINTEXT
Input:
1. Deploy application via `deploy-job`
2. Run health checks via `smoke-test-job`

Output:

deploy-job:

"Deploying application..."
Application deployed successfully

smoke-test-job:

"Performing smoke tests..."
"Health check failed"
Pipeline fails and logs: "Health check failed; exit 1"

rollback-job (if triggered):

"Rolling back application..."
Environment rolled back to previous version
CODE

### Summary
Implementing a robust GitLab CI pipeline that includes automated smoke testing and rollback ensures that your deployments are reliable. By understanding the structure, health checks, and rollback mechanisms, you can build resilient deployment processes.

**Comparison:**

```
Stage	Job	Script
Deploy	deploy-job	echo "Deploying application..."; # Your deployment commands here
Smoke Tests	smoke-test-job	`echo "Performing smoke tests..."; curl --fail http://localhost/health-check
Rollback	rollback-job	echo "Rolling back application..."; # Use GitLab API or environment configuration tools to revert deployment
```

**Code:**

```yaml
stages:
  - deploy
  - smoke-tests
  - rollback

deploy-job:
  stage: deploy
  script:
    - echo "Deploying application..."
    - # Your deployment commands here (e.g., `docker-compose up`)

smoke-test-job:
  stage: smoke-tests
  script:
    - echo "Performing smoke tests..."
    - curl --fail http://localhost/health-check || { echo "Health check failed"; exit 1; }

rollback-job:
  stage: rollback
  script:
    - echo "Rolling back application..."
    - # Use GitLab API or environment configuration tools to revert deployment
```

**Example input / output:**

```

Input:

Deploy application via deploy-job
Run health checks via smoke-test-job

Output:

CODE
Copy
deploy-job:
- "Deploying application..."
- Application deployed successfully

smoke-test-job:
- "Performing smoke tests..."
- "Health check failed"
- Pipeline fails and logs: "Health check failed; exit 1"

rollback-job (if triggered):
- "Rolling back application..."
- Environment rolled back to previous version
```

**Data:**

```yaml
# Example health check response
{
  "status": "UP",
  "details": {
    "db_connection": "OK"
  }
}
```

**Flow:**

1. Deploy
2. SmokeTests
3. SmokeTests_fail
4. RollBack
5. SmokeTests_pass
6. ProdDeployment

