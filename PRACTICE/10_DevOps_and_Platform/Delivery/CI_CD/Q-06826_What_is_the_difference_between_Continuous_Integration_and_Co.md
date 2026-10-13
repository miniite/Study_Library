---
type: "practice"
difficulty: "Basic"
---

# What is the difference between Continuous Integration and Continuous Delivery?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ CI vs Continuous Delivery vs Deployment](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_vs_Continuous_Delivery_vs_Deployment.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Continuous Integration (CI) and Continuous Delivery (CD) are both integral parts of modern software development practices, but they serve different purposes in the release pipeline. TL;DR: CI focuses on merging code changes from multiple developers into a shared repository frequently, while CD involves automatically deploying those changes to production-like environments.

To dive deeper, let's start by defining each concept and then explore their key differences.

Continuous Integration (CI)

Continuous Integration is about integrating code changes from multiple contributors into a shared mainline as early as possible. The core idea here is that frequent integration helps detect issues sooner and makes them easier to resolve in small increments. This practice encourages developers to commit code frequently, with the expectation that it will be merged back into the main development branch.

Continuous Delivery (CD)

Continuous Delivery builds on CI by adding the automation of deployment processes to a production-like environment. The key difference is that while CI ensures the code can be built and tested automatically at any time, CD focuses on making sure that changes are continuously available for release to production with minimal human intervention. This means developers have confidence in pushing their code out into production whenever they want.

#### Triage
Understanding the Probing

When an interviewer asks about CI vs CD, they're likely probing your understanding of how these practices fit into a broader DevOps strategy and how they differ in terms of automation, testing, and deployment. Assumptions here are that you recognize both practices as part of modern software development but can articulate their specific roles.

#### First-Principles Solution

CI: Focuses on merging code changes from multiple contributors into a shared mainline frequently.

Automates the build and test processes to ensure that developers catch issues early in the development cycle.
Common tools include Jenkins, GitLab CI, GitHub Actions, etc.

CD: Involves automatically deploying code to production-like environments after passing all automated tests.

Ensures that changes are ready for release by simulating a full deployment process.
Common tools include Spinnaker, AWS CodePipeline, and Kubernetes deployments.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, CI is about frequent integration to catch issues early, while CD automates the deployment process to make releases seamless. A common mistake developers might make is conflating these practices; they are distinct but complementary parts of a successful release pipeline.

#### Follow-ups
could include discussing how both practices work together in an end-to-end DevOps workflow, or exploring specific tools and their configurations for CI/CD pipelines.

**Code:**

```
MERMAID
Copy
graph TD;
    A[CI - Automate Build & Test] --> B(Continuous Integration);
    C[CD - Deploy to Prod-like Env] --> D(Continuous Delivery);
    B --> E(Automated Tests);
    D --> F(Deploy to Production);
```

**Example input / output:**

```

Example CI/CD Workflow:

Input	Output
Developer Commits	Build Passes
	Unit Tests Run
	Integration Tests Run
	Code Deployed to Staging

Example CD Configuration:

YAML
Copy
# Example Spinnaker CD Pipeline Configuration
pipeline {
    stages {
        stage('Build') {
            steps {
                echo 'Building the application...'
            }
        }
        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                echo "Deploying to ${env.STAGING_URL}"
            }
        }
        stage('Promote to Prod') {
            when {
                manual()
            }
            steps {
                echo "Promoting to Production..."
            }
        }
    }
}
```

**Data:**

```yaml
ciConfig:
  branches:
    main: true
    develop: false
cdConfig:
  environments:
    - name: staging
      url: https://staging.example.com/
    - name: production
      url: https://example.com/
```

