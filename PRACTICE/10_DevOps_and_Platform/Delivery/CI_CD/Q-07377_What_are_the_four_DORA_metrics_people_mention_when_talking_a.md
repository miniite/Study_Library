---
type: "practice"
difficulty: "Intermediate"
---

# What are the four DORA metrics people mention when talking about delivery, at a high level?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ DORA Metrics](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/DORA_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The four DORA (Deployment Frequency, Lead Time for Changes, Change Failure Rate, and Time to Restore) metrics are commonly discussed in DevOps contexts to measure how effectively a team is delivering software changes. At an intermediate level of experience, it's important to understand the core concepts behind each metric and why they matter.

**Comparison:**

|Metric| Description|
|---|---|
|Deployment Frequency| Number of times code is deployed to the production environment per month.|
|Lead Time for Changes| Average time, in hours, from when a change is made until it reaches the production environment after passing all necessary tests and approvals.|
|Change Failure Rate| Percentage of deployments that result in issues or failures once they are live in the production environment.|
|Time to Restore| Average time, in minutes, required to get services back online after a failure has occurred in the production environment.|

**Code:**

```text
# Example high-level CI/CD pipeline stages
stages:
  - build
  - test
  - deploy

build_job:
  stage: build
  script:
    - echo "Building code..."

test_job:
  stage: test
  script:
    - echo "Running tests..."

deploy_job:
  stage: deploy
  script:
    - echo "Deploying to production..."
```

**Example input / output:**

|Metric| Description|
|---|---|
|Deployment Frequency| Number of times code is deployed to the production environment per month.|
|Lead Time for Changes| Average time, in hours, from when a change is made until it reaches the production environment after passing all necessary tests and approvals.|
|Change Failure Rate| Percentage of deployments that result in issues or failures once they are live in the production environment.|
|Time to Restore| Average time, in minutes, required to get services back online after a failure has occurred in the production environment.|

**Data:**

```json
{
  "metrics": {
    "deployment_frequency": "Number of deployments per month",
    "lead_time_for_changes": "Average time (in hours) it takes for changes to go from code commit to production deployment",
    "change_failure_rate": "Percentage of deployments that cause issues in production",
    "time_to_restore": "Average time (in minutes) it takes to restore service after a failure"
  }
}
```

**Flow:**

1. Metrics
2. CI/CD Pipeline
3. Build
4. Test
5. Deploy
6. Monitor & Restore
7. Metric 1: DeploymentFrequency
8. Metric 2: Lead Time forChanges
9. Metric 3: Change Failure Rate
10. Metric 4: Time to Restore

