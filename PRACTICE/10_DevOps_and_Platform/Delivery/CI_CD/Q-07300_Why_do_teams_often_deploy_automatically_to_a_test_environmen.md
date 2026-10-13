---
type: "practice"
difficulty: "Basic"
---

# Why do teams often deploy automatically to a test environment but require a click for production?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Introducing CI CD to Legacy Systems](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Introducing_CI_CD_to_Legacy_Systems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why teams might deploy code automatically to a test environment but require manual approval for a production deployment, we need to dive into the core principles of Continuous Integration and Continuous Deployment (CI/CD). The key difference lies in how these environments are managed—test environments are often automated due to their nature, while production environments require human oversight.

TL;DR

Teams deploy automatically to test environments because they mimic real-world conditions closely, enabling automated testing. In contrast, production deployments require manual approval to ensure rigorous validation and avoid the risks associated with live systems.

#### Triaging the Question

The question revolves around the difference in deployment approaches for test and production environments within a CI/CD pipeline. The core of the inquiry is why there's an automatic process for tests but a deliberate step involving human intervention before going live.

Key Assumptions:
Test Environment: This environment is designed to closely mirror the production setup, often with similar data and configurations.
Production Environment: This environment contains real user data and business-critical applications, making it essential to ensure stability and safety.
Step-by-Step Solution
1. Test Environments

In a test environment, automation plays a significant role because:

#### Cons
istency: Tests are run in an identical setup as the production environment, ensuring that any discrepancies are caught before going live.
Speed and Efficiency: Automated tests can be run continuously, providing faster feedback on code changes.
Resource Utilization: Test environments are often spun up temporarily or reused, making them cost-effective.
2. Production Environments

For production deployments, manual approval is necessary because:

Complexity and Risk: The consequences of a failure in the production environment can be severe, affecting real users and potentially causing business disruptions.
Human Oversight: Manual checks ensure that critical steps are not missed and that changes go through rigorous testing.
Security Considerations: Production environments often contain sensitive data and require strict access controls. Manual approval helps enforce these policies.
3. Balancing Automation and Human Touch

The goal is to strike a balance between automation (for efficiency and consistency) and human oversight (to ensure safety). This can be achieved through:

Staged Deployments: Implementing multi-stage deployment pipelines where each stage might have different levels of automation.
Automated Checks with Manual Approval: Running automated checks but requiring manual approval to proceed, ensuring that any last-minute issues are caught before going live.
4. Practical Example

#### Cons
ider a typical CI/CD pipeline:

CI: Automated build and test phases run every time code is committed.
CD to Test Environment: Deployment automatically occurs in the test environment if all tests pass.
Manual Approval for Production: Before deploying to production, an automated check (like a pre-deployment script) is followed by manual approval.

#### Closing Summary

In summary, automatic deployments to test environments and manual approvals before production releases are strategic decisions driven by the need for consistency in testing and rigorous validation in production. Automation speeds up development cycles and reduces human errors, while manual steps ensure that critical checks are not skipped, maintaining high levels of system reliability and security.

#### Common Mistakes
and Follow-Ups
Over-Automating Production: Failing to implement any form of human oversight can lead to undetected bugs or security vulnerabilities.
Under-Automizing Tests: Relying too heavily on manual testing can slow down the development process and introduce inconsistencies.
Ignoring Security Protocols: Not enforcing strict access controls and security checks can pose significant risks.

If asked further, you might explore specific tools like Kubernetes Deployments, Jenkins pipelines, or GitOps strategies that enable controlled automation with manual gates in place. Understanding these concepts will help in designing robust CI/CD practices tailored to your organization's needs.

**Comparison:**

```
Environment	Deployment Type	Automation Level
Test	Automated	High
Prod	Manual Approval	Low
```

**Code:**

```yaml
# Example of a simple Jenkins pipeline for automated testing and manual production deployment

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    // Automated build steps here
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Run automated tests
                }
            }
        }

        stage('Deploy to Test') {
            when {
                branch 'main'
            }
            steps {
                script {
                    // Deploy automatically to test environment
                }
            }
        }

        stage('Pre-Production Check') {
            when {
                branch 'main'
            }
            steps {
                script {
                    // Pre-deployment checks, e.g., security scan
                }
            }
        }

        stage('Deploy to Production (Manual Approval)') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Please approve the production deployment.'
            }
        }
    }
}
```

**Example input / output:**

```text
Input:
- Code changes in a branch 'main'
- Automated tests pass

Output:
1. Build and test phases automatically run.
2. Deployment to the test environment occurs.
3. Pre-deployment checks are run.
4. User is prompted for manual approval before deployment to production.

Dry Run:
- A change in the 'main' branch triggers a build and tests (pass).
- Test environment deployed successfully.
- No issues detected, so user approves the release.
- Production environment updated manually after approval.
```

**Data:**

```json
{
  "testEnv": {
    "url": "http://test.example.com",
    "configurations": [
      {"name": "database", "type": "mysql"},
      {"name": "api", "type": "rest"}
    ]
  },
  "prodEnv": {
    "url": "http://prod.example.com",
    "configurations": [
      {"name": "database", "type": "sensitive"},
      {"name": "api", "type": "rest"}
    ]
  }
}
```

**Flow:**

1. Code Change
2. CI/CD Pipeline
3. Automated Build & Tests
4. Deploy to Test Env. (Auto)
5. Pre-Deployment Checks
6. Manual Approval for Prod.
7. Deploy to Prod. Env. (Auto)

