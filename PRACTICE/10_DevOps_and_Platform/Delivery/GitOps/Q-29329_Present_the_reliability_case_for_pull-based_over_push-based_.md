---
type: "practice"
difficulty: "Deep"
---

# Present the reliability case for pull-based over push-based delivery to a skeptical security team — what threat-model advantages does the pull model buy, and where does it fall short?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Sync Notifications](<../../../../10_DevOps_and_Platform/Delivery/GitOps/GitOps_Fundamentals/Sync_Notifications.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're discussing whether to adopt a pull-based GitOps approach over a push-based one from a reliability perspective, particularly in terms of security concerns. A pull-based model ensures that changes are validated and tested before being applied, which can provide significant security benefits. However, it also has its limitations.

Let's break down the advantages and shortcomings of pull-based delivery to address the skepticism of a security team.

#### Triage
What the Question is Really Asking

The core question revolves around the security trade-offs between push- and pull-based GitOps approaches. The interviewer likely wants you to:

Explain the reliability and security benefits of pull-based models.
Identify potential vulnerabilities in both approaches.
Provide a balanced view, acknowledging both strengths and weaknesses.

#### Assumptions
We're assuming that the security team is concerned about unauthorized deployments or breaches.
Both push and pull models involve automation but differ in how they validate changes before deployment.
First Principles: Pull-Based GitOps

In a pull-based GitOps model, changes are validated by a continuous integration/continuous delivery (CI/CD) pipeline. This means that any modification to the codebase must pass through automated tests and approvals before being deployed to production. Here’s how this approach addresses security concerns:

Code Validation: Pull requests trigger a CI/CD pipeline that performs static analysis, unit tests, integration tests, and other validations. This ensures that only reliable and secure code makes it into the main branch.
Change Approval: Team members must approve changes before they can be merged, which adds an extra layer of human scrutiny and security review.
Environment Isolation: Changes are deployed to a staging environment first for additional testing and verification before being promoted to production. This helps catch any issues early on.
Common Threat Models
Unauthorized Deployment: In a push-based model, a developer could bypass the validation process and deploy code directly to production, potentially introducing security vulnerabilities.
Human Error or Malice: Even with strict policies, human errors can still occur. A pull request system reduces this risk by requiring multiple reviews.
Where Pull-Based Delivery Falls Short
Latency in Deployment: In highly dynamic environments where rapid deployment is critical, the additional validation steps required by a pull-based model can introduce latency.
Complexity in Rollbacks: If an issue is detected post-deployment, rollbacks become more complex and time-consuming because changes need to be reverted through the same CI/CD pipeline.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Pull-based GitOps provides robust security benefits by ensuring that code undergoes rigorous validation before deployment. However, it may introduce additional delays in high-velocity scenarios. The key is finding a balance where security is prioritized without compromising on the speed of deployments. Common mistakes include underestimating the overhead of pull requests or overemphasizing their benefits without considering real-world use cases.

When addressing this with the security team:

Highlight specific security features like CI/CD pipelines and code reviews.
Discuss potential bottlenecks in terms of deployment time and rollback complexity.
Emphasize the importance of a well-defined pull request process and its role in reducing risks.

**Comparison:**

```
Threat Model	Push-Based	Pull-Based
Unauthorized Deployment	High risk if developer bypasses validation	Low risk due to CI/CD pipeline
Human Error or Malice	Medium risk with strict policies but human error still possible	Low risk due to multiple reviews
```

**Code:**

```
MERMAID
Copy
graph TD
    A[Code Repository] --> B[CI/CD Pipeline]
    B --> C[Staging Environment]
    C --> D[Production Environment]
    E[Manual Approval] --> B
```

**Example input / output:**

```text
Request:
- Developer submits a pull request with changes to fix a security vulnerability.

Output:
- Pull request triggers CI/CD pipeline.
- Pipeline performs static analysis, unit tests, and integration tests.
- All tests pass; system merges pull request into main branch.
- Changes are deployed to staging environment for further testing.
```

**Data:**

```json
{
  "pullRequest": {
    "status": "open",
    "author": "John Doe",
    "changes": ["fix security vulnerability", "update dependencies"]
  },
  "validationSteps": [
    { "step": "static code analysis", "result": "passed" },
    { "step": "unit tests", "result": "passed" },
    { "step": "integration tests", "result": "passed" }
  ]
}
```

**Flow:**

1. Code Change
2. Pull Request
3. CI/CD Pipeline
4. Staging Env
5. Production Env
6. Manual Approval

