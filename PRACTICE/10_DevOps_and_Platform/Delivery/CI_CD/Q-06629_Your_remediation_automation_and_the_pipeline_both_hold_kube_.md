---
type: "practice"
difficulty: "Deep"
---

# Your remediation automation and the pipeline both hold kube credentials; how would you enforce least privilege so the pipeline can deploy the runbook but only the runtime can execute cluster actions?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Secrets in Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce least privilege in a CI/CD context where both remediation automation and the pipeline hold Kubernetes (kube) credentials, we need to ensure that the pipeline has minimal permissions to deploy runbooks, while the runtime can only execute cluster actions necessary for executing those runbooks. This involves creating specialized service accounts with restricted roles, leveraging role-based access control (RBAC), and implementing a robust credential management strategy.

The core idea is to minimize the attack surface by granting specific permissions at each step of the pipeline. We'll start by defining the required permissions for each component and then configure them accordingly.

