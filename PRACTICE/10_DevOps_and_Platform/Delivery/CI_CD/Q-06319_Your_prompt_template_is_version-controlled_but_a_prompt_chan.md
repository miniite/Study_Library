---
type: "practice"
difficulty: "Deep"
---

# Your prompt template is version-controlled but a prompt change ships with no test; how would you add an offline eval stage that treats a prompt edit as a deployable artifact requiring the same gates as code?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Build and Release Metadata](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Build_and_Release_Metadata/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with AI prompts in a CI/CD pipeline, treating them as deployable artifacts and adding an offline evaluation stage is crucial to ensure that changes are thoroughly tested before deployment. This approach mirrors the rigor applied to code reviews and testing phases, ensuring quality and reliability.

The question asks us to design an offline eval stage for prompt edits that requires the same gates (like approvals, tests, etc.) as code changes. Here's how we can achieve this:

Understand the Requirement: We need to treat a prompt edit as if it were a code change, which means incorporating it into our CI/CD pipeline and ensuring it undergoes the same quality checks.

Identify Key Gates: These typically include automated tests, manual reviews, approvals from relevant stakeholders, and deployment readiness checks.

Design the Flow:

Commit Prompt Changes: Any change to a prompt should be committed as part of a code commit.
Build Pipeline: This step compiles or packages the updated prompt along with any necessary dependencies.
Test Stage: Automated tests are run to ensure the new prompt doesn't introduce bugs or regressions. These can include unit tests, integration tests, and end-to-end tests (if applicable).
Review and Approve: A review process is initiated where domain experts or stakeholders approve the changes.
Offline Evaluation: An offline eval stage runs the updated prompt in a controlled environment to simulate real-world usage without affecting production systems. This could involve running the prompt through various scenarios to ensure it behaves as expected.
Promote to Production: Once all gates are passed, the updated prompt is deployed to production.

#### Implementation
Details:

Version Control for Prompts: Maintain version control for prompts just like code. Use Git or a similar system to track changes and manage branches.
CI/CD Integration: Integrate the prompt changes into the CI pipeline so that every commit triggers an automated build, test, and evaluation process.
Feedback Loop: Establish a clear feedback loop between the offline eval stage and the development environment. If issues are found during the eval, the prompt should be reverted to the previous version or updated further based on the findings.

Monitoring and Metrics:

Monitor Performance: After deployment, continuously monitor the performance of the new prompt using metrics like success rates, error rates, and user feedback.
Feedback Mechanism: Implement a mechanism for users to provide feedback directly in production if they encounter issues with the updated prompt.

Edge Cases and Failures:

Handling Rejected Changes: If changes are rejected during review or testing, ensure there’s a process to revert to the previous version seamlessly.
Rollback Plan: Have a rollback plan ready in case an issue is detected post-deployment that wasn’t caught by tests.

**Comparison:**

```
Stage	Description
Prompt Changes	Commit prompt changes and merge to main branch
Build Pipeline	Package or compile updated prompts
Automated Tests	Run automated tests for regressions
Code Review	Submit for review and get approvals
Offline Eval	Run offline eval in a controlled env.
Promote to Prod	Deploy updated prompt to production
```

**Code:**

```python
# Example of integrating prompt changes into CI pipeline using Jenkinsfile

pipeline {
    agent any

    stages {
        stage('Prompt Changes') {
            steps {
                // Commit prompt changes and merge to main branch
                script {
                    git 'https://github.com/yourrepo/prompts'
                    sh 'git commit -am "Updated prompt [version]" && git push origin main'
                }
            }
        }

        stage('Build Pipeline') {
            steps {
                // Build package or compile updated prompt code
                sh './build.sh'
            }
        }

        stage('Automated Tests') {
            steps {
                // Run automated tests to ensure no regressions
                sh 'pytest test_prompts.py'
            }
        }

        stage('Code Review and Approve') {
            steps {
                // Submit for review and get approvals
                script {
                    sh 'git pull-request --message "Updated prompt [version] - reviewed by [reviewer]"'
                }
            }
        }

        stage('Offline Evaluation') {
            steps {
                // Run offline eval in a controlled environment
                sh 'python evaluate_prompts.py'
            }
        }

        stage('Promote to Production') {
            when {
                expression { return params.PROMOTE_TO_PRODUCTION == true }
            }
            steps {
                // Deploy updated prompt to production
                sh 'kubectl apply -f prod-prompts.yaml'
            }
        }
    }

    post {
        always {
            echo "Pipeline complete"
        }

        success {
            echo "Promotion successful"
        }

        failure {
            echo "Promotion failed, rolling back changes"
            script {
                sh 'git reset --hard HEAD~1 && git push origin main'
            }
        }
    }
}
```

**Example input / output:**

```text
# Example of an input-output for the offline eval stage

Input: Prompt "Welcome to our service!"
Output:
- Scenario 1: User response "Hi"
  Expected Response: "Hello, how can I assist you today?"
  Actual Response: "Hello, how can I assist you today?"

- Scenario 2: User response "I need help with..."
  Expected Response: "Sure, could you please specify the area where you need assistance?"
  Actual Response: "Sure, could you please specify the area where you need assistance?"

# Evaluation Summary
All scenarios passed successfully. No unexpected behavior detected.
```

**Data:**

```yaml
# Example prod-prompts.yaml file for deployment

apiVersion: v1
kind: Deployment
metadata:
  name: prompt-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: prompt-service
    spec:
      containers:
      - name: prompt-container
        image: yourpromptimage:v2.0.0
        ports:
        - containerPort: 8080
```

**Flow:**

1. Success
2. Failure
3. Commit Prompt Changes
4. Build Pipeline
5. Automated Tests
6. Code Review and Approve
7. Offline Evaluation
8. Promote to Production
9. Promotion Successful
10. Rollback Changes

