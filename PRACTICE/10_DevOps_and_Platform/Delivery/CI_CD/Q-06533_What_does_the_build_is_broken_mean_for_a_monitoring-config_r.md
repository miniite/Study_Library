---
type: "practice"
difficulty: "Basic"
---

# What does 'the build is broken' mean for a monitoring-config repo, and who is usually expected to fix it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Schema Changes in Delivery Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Schema_Changes_in_Delivery_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When we say "the build is broken" in the context of a monitoring-config repository within an AIOps environment, it means that there has been a change or issue introduced into the configuration codebase that prevents the automation pipeline from successfully building and deploying the configurations. This could be due to a syntax error, incorrect dependencies, or any other problem that causes the build process to fail.

The question is probing our understanding of CI/CD practices and roles within an AIOps team. Typically, when such a failure occurs, the person responsible for maintaining the pipeline or the developer who introduced the change are expected to address it. Let's break down the key concepts involved:

#### Triaging the Question

What "the build is broken" means:

The term refers to an issue in the CI/CD pipeline that causes a failure during the build stage.

Roles involved:

DevOps engineers or CI/CD specialists who maintain the pipeline.
Developers who introduce changes into the monitoring-config repository.

Expected actions:

Identifying and fixing the root cause of the issue.
Ensuring successful builds for future deployments.

#### First-Principles Solution

Identify the Issue:

Review the build logs to identify the specific error or failure point. This could involve checking the output of tools like Jenkins, GitLab CI, or similar CI/CD platforms.

Determine the Cause:

Determine if the issue is due to a new change in the codebase introduced by developers.
Verify that all dependencies are correctly specified and available.
Ensure that the configurations adhere to best practices for deployment and monitoring.

Fix the Issue:

If the issue is caused by a developer's changes, they should update their local environment and push corrected code.
If it’s due to an infrastructure or configuration issue, involve DevOps engineers who maintain the pipeline setup.

Rebuild and Validate:

Once the issue is resolved, rebuild the configurations and validate that the build passes without errors.
Deploy the changes to a staging environment if necessary before moving to production.

#### Closing Summary
+ Common Mistakes/Follow-Ups
The key takeaway is that "the build is broken" indicates an urgent need for troubleshooting. Developers or DevOps engineers should collaborate to identify and fix the issue quickly to maintain the integrity of the monitoring configurations.

#### Common mistakes
Ignoring the failure and hoping it resolves itself.
Not following proper logging and error handling practices in CI/CD pipelines, making it harder to diagnose issues.

#### Follow-Ups
How do you handle situations where multiple changes are introduced simultaneously?
What steps would you take if the build fails on a critical release branch?

**Comparison:**

```
Stage	Action
Build	Execute 'make build'
Test	Execute 'make test'
Deploy	Execute 'make deploy'
```

**Code:**

```python
# Example of a simple Jenkinsfile snippet for CI/CD pipeline
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
        stage('Test') {
            steps {
                sh 'make test'
            }
        }
        stage('Deploy') {
            steps {
                sh 'make deploy'
            }
        }
    }
}
```

**Example input / output:**

```text
Input:
Build failed for monitoring-config repo at 2023-10-10T14:30:00Z

Output:
Error message: "Syntax error in config.yml: expected a boolean value but found integer at line 5, column 10"
```

**Data:**

```json
{
  "build": {
    "status": "broken",
    "error_message": "Syntax error in config.yml: expected a boolean value but found integer at line 5, column 10"
  },
  "last_modified_by": "John Doe",
  "date_time": "2023-10-10T14:30:00Z"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Identify the Issue
6. Is it a developer's change?
7. Contact Developer
8. Determine if Infra or Configissue
9. Is it infrastructure related?
10. Notify DevOps Engineers
11. Fix the Issue locally and pushchanges
12. Rebuild and validate thepipeline
13. Determine root cause and fixinfra/config
14. Pipeline passes, proceed withdeployment
15. Determine root cause and fixinfra/config

