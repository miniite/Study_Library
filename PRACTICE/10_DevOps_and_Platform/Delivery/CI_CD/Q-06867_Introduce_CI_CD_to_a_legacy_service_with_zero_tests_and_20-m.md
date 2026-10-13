---
type: "practice"
difficulty: "Deep"
---

# Introduce CI/CD to a legacy service with zero tests and 20-minute manual deploys; what's your first two weeks, and how do you build a safety net before you start automating deploys?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Introducing CI CD to Legacy Systems](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Introducing_CI_CD_to_Legacy_Systems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
In this scenario, we're tasked with transitioning a legacy service from manual 20-minute deployments to an automated CI/CD pipeline. The first two weeks will focus on setting up continuous integration and deployment (CI/CD) while building a safety net to prevent regressions. This involves understanding the application, identifying critical paths, and implementing unit tests, static code analysis, and integration testing.

#### Triage
The core of this question is about establishing a CI/CD pipeline for an untested legacy service with minimal automation. The interviewer is probing your ability to quickly assess the state of the system, prioritize tasks effectively, and implement robust safety measures before full-scale automated deployments. Assumptions include having access to version control systems (e.g., Git), deployment environments, and development tools.

#### First-Principles Solution

Initial Assessment:

Spend the first day reviewing the codebase, environment setup, and existing manual processes.
Identify critical components and dependencies of the legacy service.

Setup CI/CD Infrastructure:

Choose a CI/CD platform (e.g., Jenkins, GitLab CI, GitHub Actions).
Set up the necessary configurations for version control integration.
Create initial pipeline definitions to cover basic build steps (compiling code, running static analysis).

Build Safety Net:

Prioritize writing unit tests and integration tests based on critical paths identified during assessment.
Implement a staging environment that mirrors production but is easier to reset if something goes wrong.

Implement Incremental Automation:

Start with basic deployment automation, ensuring every change can be deployed safely in the staging environment before moving to production.
Gradually expand test coverage and automate more stages of the pipeline (e.g., unit tests, integration tests).

Document and Communicate:

Document all changes made during this period for future reference.
Engage with stakeholders to ensure everyone understands the new deployment process.

#### Closing Summary
The first two weeks involve setting up a CI/CD pipeline from scratch while building a robust safety net through testing, staging environments, and incremental automation. This approach ensures that even if something goes wrong during early automated deployments, there are measures in place to catch issues before they impact production.

#### Common Mistakes/Follow-Ups

Not spending enough time on initial assessment.
Overlooking the importance of a robust safety net (e.g., not writing unit tests).
Ignoring communication with stakeholders about changes and risks associated with new deployments.
Failing to choose appropriate CI/CD tools for the specific requirements.

**Comparison:**

```
Stage	Description
Build	Compile codebase
Static Analysis	Run static code analysis tools
Unit Tests	Execute unit tests to catch bugs early
Integration Tests	Ensure components work together as expected
Deploy to Staging	Safely deploy changes to staging environment for review
Promote to Production	Automate promotion from staging to production if all tests pass
```

**Code:**

```yaml
# Example Jenkinsfile snippet
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Compiling code'
                // Add compilation commands here
            }
        }
        stage('Static Analysis') {
            steps {
                script {
                    sh 'java -jar static-analysis.jar'
                }
            }
        }
        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                echo 'Deploying to staging environment'
                // Add deployment commands here
            }
        }
    }
}
```

**Example input / output:**

```
Input: Manual deployment of a commit to main branch
Output:
CI/CD pipeline triggers build and static analysis stages.
Deploy to staging environment is successful.
No issues in unit or integration tests, deployment progresses to production.
JSON
Copy
{
  "commit_message": "Fix bug in login endpoint",
  "staging_env_results": {
    "static_analysis_passes": true,
    "unit_tests_passed": true,
    "integration_tests_passed": true
  },
  "prod_env_results": {
    "deployment_successful": true
  }
}
```

**Data:**

```json
{
  "git_repo_url": "https://github.com/example/legacy-service",
  "staging_env_vars": {
    "DB_USERNAME": "staging_user",
    "DB_PASSWORD": "staging_pass"
  },
  "prod_env_vars": {
    "DB_USERNAME": "prod_user",
    "DB_PASSWORD": "prod_pass"
  }
}
```

**Flow:**

1. Initial Assessment
2. Setup CI/CD
3. Build Safety Net
4. Implement IncrementalAutomation
5. Document & Communicate

