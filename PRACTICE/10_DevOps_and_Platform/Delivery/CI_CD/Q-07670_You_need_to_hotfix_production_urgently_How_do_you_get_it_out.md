---
type: "practice"
difficulty: "Deep"
---

# You need to hotfix production urgently. How do you get it out safely through CI/CD?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Emergency Deploy Paths](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Emergency_Deploy_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address a critical production issue with an urgent hotfix, we must leverage our Continuous Integration and Continuous Deployment (CI/CD) pipeline to ensure safety and reliability. The key steps involve isolating the problem, identifying the fix, integrating it into the codebase, testing thoroughly, and then deploying safely.

The interviewer is probing your understanding of CI/CD principles, deployment strategies, and best practices for handling urgent fixes without disrupting production. Assumptions include a well-defined CI/CD pipeline with multiple stages like unit tests, integration tests, and possibly security scans before reaching the production environment.

Step-by-Step Solution

Identify the Issue:

Use monitoring tools (e.g., Prometheus, Grafana) to pinpoint the exact error or issue in production.
Gather logs from the affected components and ensure that you have a clear understanding of the root cause.

Develop the Fix:

Create a new branch for the hotfix in your version control system (VCS).
Apply the fix directly on this branch, ensuring it’s concise and focused.
Perform unit tests to validate the changes locally.

Integrate the Fix:

Push the fix from the hotfix branch to the CI/CD pipeline.
The pipeline should automatically trigger a series of stages:
Static Code Analysis: Tools like SonarQube ensure no regressions or code smells.
Unit Tests: Automatically run your unit tests to catch any issues locally.
Integration Tests: Test the fix in isolation from other systems if necessary.
Security Scans: Use tools like Snyk to check for vulnerabilities.

Manual Testing:

If the CI/CD pipeline passes, manually verify the fix on a staging environment that closely mimics production.
This helps catch any issues that may not have been identified in automated tests.

Deploy Safely:

Once satisfied with manual testing, proceed to deploy to production using a canary release strategy:
Gradually roll out the fix to a small percentage of users (e.g., 1%).
Monitor these users closely for any issues.
If all goes well, scale up gradually.

Rollback Plan:

Ensure you have a rollback plan in place if the hotfix introduces new issues.
This can involve reverting to the previous commit or rolling back using version control operations.

Post-Deployment Monitoring:

Continuously monitor the system for any anomalies post-deployment.
Log all changes and fixes, and document the process for future reference.

#### Closing Summary

Handling an urgent hotfix safely requires a disciplined approach that leverages CI/CD practices to ensure quality and reliability. By isolating the issue, testing thoroughly, and deploying carefully, you can minimize risks and maintain production uptime.

#### Common Mistakes/Follow-Ups
Skipping Automation: Not running all necessary checks through the pipeline can introduce issues.
Ignoring Monitoring: Failing to set up proper monitoring after deployment can lead to unnoticed problems.
Over-Rolling Changes: Making large changes without thorough testing can compound issues.
No Rollback Plan: Lack of a rollback strategy means no way to revert if something goes wrong.

**Comparison:**

```
Stage	Description
Static Code Analysis	Run static code analysis tools to check for issues.
Unit Tests	Execute unit tests automatically using CI/CD.
Integration Tests	Test integration with other systems if necessary.
Security Scans	Perform security scans to ensure no vulnerabilities.
Manual Testing	Manually verify the fix on a staging environment.
Deployment	Gradually roll out the fix to production using canary release.
```

**Code:**

```bash
# Example Jenkins pipeline snippet for hotfix deployment

pipeline {
    agent any

    environment {
        DEPLOY_ENV = 'production'
    }

    stages {
        stage('Static Code Analysis') {
            steps {
                script {
                    sh 'sonar-scanner'
                }
            }
        }
        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Integration Tests') {
            steps {
                sh 'java -jar integration-tests.jar'
            }
        }
        stage('Security Scans') {
            steps {
                script {
                    sh 'snyk scan project'
                }
            }
        }
        stage('Manual Testing') {
            when {
                expression { return params.isStaging() }
            }
            steps {
                echo "Manually tested on staging environment"
            }
        }
    }

    post {
        always {
            script {
                // Log changes and document the process
                sh 'git log --oneline'
            }
        }
        success {
            // Proceed to deployment
            echo "Deployment successful, proceeding to production."
        }
        failure {
            // Handle rollback
            echo "Rolling back to previous version..."
        }
    }
}
```

**Example input / output:**

```text
Input:
- Issue in production identified by monitoring tools.
- Hotfix developed and pushed to CI/CD pipeline.

Output:
1. Static Code Analysis passed.
2. Unit Tests passed.
3. Integration Tests passed.
4. Security Scans clear.
5. Manual testing on staging environment successful.
6. Canary release strategy deployed 1% of users.
7. Full deployment to production.
8. Monitoring post-deployment with no issues detected.

Dry-Run:
- Staging Environment Test Results: All tests pass, no regressions.
- Production Rollout Plan: Gradually scale from 1% to 100% over next 3 hours.
```

**Data:**

```json
{
  "git_branch": "hotfix-1234",
  "staging_environment_url": "http://staging.example.com",
  "production_environment_url": "http://prod.example.com"
}
```

**Flow:**

1. Identify Issue
2. Collect Logs
3. Diagnose Root Cause
4. Develop Hotfix
5. Integrate into CI/CD
6. Trigger Pipeline
7. Static Code Analysis
8. Unit Tests
9. Integration Tests
10. Security Scans
11. Manual Testing (Staging)
12. Canary Release
13. Gradual Deployment to Prod
14. Post-Deployment Monitoring
15. Rollback Plan in Place

