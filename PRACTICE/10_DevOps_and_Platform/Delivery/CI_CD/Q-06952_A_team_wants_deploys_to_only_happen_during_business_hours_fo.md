---
type: "practice"
difficulty: "Intermediate"
---

# A team wants deploys to only happen during business hours for safety, but that batches risk into fewer, bigger releases; make the argument for continuous deploys with strong guardrails instead, backed by data you'd collect.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Standard Deployment Strategies](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/Standard_Deployment_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're tasked with making a compelling case for adopting continuous deployment (CD) while ensuring robust safety measures are in place. The core argument is that continuous deploys, when properly secured, can actually reduce overall risk and improve the quality of deployments by catching issues earlier.

#### Triaging the Question

The question asks us to justify why CD should be preferred over batched releases during business hours, given that batched releases allow fewer but potentially higher-risk changes. We need to consider several factors: risk management, detection mechanisms, operational overhead, and the impact on system reliability and customer satisfaction.

#### First-Principles Solution

The key idea here is to implement a robust CI/CD pipeline with stringent guardrails, ensuring that every deployment goes through thorough automated tests before being pushed live. This approach leverages the benefits of continuous integration (CI) while mitigating the risks associated with frequent deployments.

Automated Testing: Implement comprehensive unit, integration, and end-to-end tests to catch issues early.
Staging Environments: Use staging environments that mirror production closely to detect any unexpected behavior or performance issues.
Rollback Mechanisms: Ensure quick rollback capabilities in case something goes wrong.
Monitoring and Logging: Continuous monitoring and logging for real-time issue detection post-deployment.
Security Checks: Regular security scans and compliance checks.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In conclusion, continuous deploys with strong guardrails provide a safer alternative to batched releases. The key is not just the frequency but also the thoroughness of the safety mechanisms in place. By leveraging modern CI/CD tools and practices, we can reduce risk while maintaining high-quality deployments.

#### Common mistakes
include:

Overlooking rigorous testing phases.
Failing to set up adequate rollback procedures.
Ignoring monitoring and logging post-deployment.

#### Follow-ups
might involve discussing specific tools for implementation, like Jenkins, GitLab CI, or Kubernetes-based pipelines, as well as how these tools can integrate with existing DevOps practices.

**Comparison:**

```
Stage	Description	Tools Used
Build	Compiling and packaging code	Maven, Gradle
Test	Executing tests (unit, integration, end-to-end)	JUnit, Selenium, Postman
Deploy	Pushing changes to staging environment	Jenkins, GitLab CI/CD
```

**Code:**

```yaml
# Example YAML for a simple CI/CD pipeline using GitLab
stages:
  - build
  - test
  - deploy

variables:
  MAVEN_OPTS: "-Dmaven.test.failure.ignore=true"

deploy-staging-job:
  stage: deploy
  script:
    - echo "Deploying to staging environment..."
    - mvn clean install deploy --settings settings.xml -DskipTests
  dependencies: []
  only:
    - master

test-job:
  stage: test
  script:
    - mvn clean verify
  artifacts:
    paths:
      - target/*.jar
  dependencies: [deploy-staging-job]
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Deploy to staging environment with `mvn clean install deploy --settings settings.xml -DskipTests`<br>Run automated tests using `mvn clean verify` | Staging deployment successful. Tests passed.<br>No issues detected in the staging environment. Proceeding to production after manual review. |

**Data:**

```json
{
  "stages": ["build", "test", "deploy"],
  "variables": {
    "MAVEN_OPTS": "-Dmaven.test.failure.ignore=true"
  },
  "deploy-staging-job": {
    "stage": "deploy",
    "script": [
      "echo 'Deploying to staging environment...'",
      "mvn clean install deploy --settings settings.xml -DskipTests"
    ],
    "dependencies": [],
    "only": ["master"]
  },
  "test-job": {
    "stage": "test",
    "script": ["mvn clean verify"],
    "artifacts": {"paths": ["target/*.jar"]},
    "dependencies": ["deploy-staging-job"]
  }
}
```

**Flow:**

1. Code Changes
2. Commit to Repo
3. Branch Merge Request
4. Automated Build & Test
5. Pass -> Staging Deployment,Fail -> Code Fixes
6. Manual Review -> Approve ->Production Deployment, Reject-> Back to Code Fixes

