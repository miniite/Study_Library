---
type: "practice"
difficulty: "Deep"
---

# How does artifacts:expire_in interact with keep-latest-artifacts and storage cost at scale?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cache vs Artifacts](<../../../../TOOLS/GitLab/GitLab_CI/Cache_vs_Artifacts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Artifacts expiration in GitLab CI, when combined with keep_latest_artifacts, can significantly impact storage costs at scale. The artifacts:expire_in directive sets a retention period for artifacts while keep_latest_artifacts ensures only the latest successful run's artifacts are stored. Understanding how these interact and their implications on storage cost is crucial for managing CI/CD pipelines effectively.

First, let’s break down what each of these settings means:

artifacts:expire_in: This directive sets a time limit after which artifacts will be automatically deleted by GitLab. For example, if you set expire_in: 1 week, any artifact older than one week will be removed.
keep_latest_artifacts: This setting ensures that only the latest successful run's artifacts are kept, even if they would otherwise expire based on the artifacts:expire_in directive.

When these settings interact:

If a new pipeline runs and keep_latest_artifacts is set to true, GitLab will keep the artifacts from this latest run.
However, if an artifact that was part of a previous successful run is older than the artifacts:expire_in period, it will be deleted regardless of whether keep_latest_artifacts is active.

In essence, keep_latest_artifacts can override the automatic deletion based on artifacts:expire_in, but only for artifacts from the latest successful run. This interaction helps manage storage costs by ensuring that old successful runs are not unnecessarily retained while still keeping the most recent data available.

The key points to consider when using these settings at scale include:

Storage Cost Management: Artifacts can accumulate over time, especially if you have a large number of pipelines and frequent builds. Setting appropriate expire_in values helps manage storage costs by ensuring that older artifacts are automatically cleaned up.
Purge Policies: GitLab CI allows for fine-grained control through purge policies, which complement artifacts:expire_in. These can be used to delete old artifacts more aggressively if necessary.

Here’s a practical example: Suppose you have 10 pipelines, each running every hour. Setting artifacts:expire_in: 24 hours and enabling keep_latest_artifacts would ensure that only the most recent successful run's artifacts are stored, with older ones being deleted automatically after one day.

To summarize, artifacts:expire_in and keep_latest_artifacts interact in a way that ensures both automatic cleanup of old artifacts and retention of the latest successful runs. This combination helps in managing storage costs effectively while maintaining access to critical recent data.

**Comparison:**

| Job  | Artifacts Expire In | Keep Latest Artifacts? | Result |
|------|---------------------|------------------------|--------|
| Job1 | 24 hours            | No                     | Old artifacts deleted after 24 hours, latest successful runs retained. |
| Job2 | 1 week              | Yes                    | All runs kept, but old ones will be deleted if older than 1 week except the latest success. |

**Code:**

```yaml
# Example GitLab CI pipeline configuration
stages:
  - build

job1:
  stage: build
  script:
    - echo "Running job1"
  artifacts:
    expire_in: 24 hours # Artifacts will be deleted after 24 hours unless they are the latest.
    paths:
      - output/
    when: on_success # Only keep successful runs

job2:
  stage: build
  script:
    - echo "Running job2"
  artifacts:
    expire_in: 1 week # Artifacts will be deleted after one week, except the latest success.
    paths:
      - output/
    when: always # Keep all runs
```

**Example input / output:**

| Input                         | Output                       |
|-------------------------------|------------------------------|
| Pipeline runs every hour with `artifacts:expire_in` set.       | Artifacts older than the specified period are automatically deleted, except for the latest successful run of each job. |

**Data:**

```json
{
  "artifacts": {
    "job1": {
      "expire_in": "24 hours",
      "latest_success_only": true
    },
    "job2": {
      "expire_in": "1 week",
      "keep_latest_artifacts": true,
      "paths": ["output/"]
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Job Runs
6. Artifacts Expire In Set?
7. Old Artifacts Deleted
8. Keep Latest Artifacts Set?
9. Latest Success ArtifactsRetained, Old Deleted
10. All Runs Kept

