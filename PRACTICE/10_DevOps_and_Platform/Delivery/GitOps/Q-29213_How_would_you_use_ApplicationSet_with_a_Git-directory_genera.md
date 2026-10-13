---
type: "practice"
difficulty: "Deep"
---

# How would you use ApplicationSet with a Git-directory generator to onboard new services automatically while still enforcing per-team RBAC?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Multi-tenant GitOps](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Repo_and_Environment_Layout/Multi-tenant_GitOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To onboard new services using ApplicationSet in conjunction with a Git-directory-generator, we need to establish an automated workflow that ensures compliance with team-based role-based access control (RBAC) policies. This involves setting up the Git-directory-generator to dynamically generate Git repositories for each service, integrating it with the ApplicationSet manifest generator, and configuring the necessary RBAC rules in our cluster’s Kubernetes API server.

#### Triage
The question is asking how to automate the onboarding process of new services using GitOps practices while enforcing fine-grained access controls. Specifically, we need to leverage a Git-directory-generator to manage directory structures within Git repositories and use an ApplicationSet to define the application deployment logic. We must also ensure that this setup adheres to RBAC policies defined by teams.

#### First-Principles Solution

Directory Structure Generation: Utilize a Git-directory-generator tool, such as Pulumi or Terraform, to create Git repositories for each new service based on specific templates. This tool will automatically generate the necessary directory structures and files (like Chart.yaml, values.yaml, etc.) required by Helm charts.

ApplicationSet Manifest Generation: Define an ApplicationSet manifest using a tool like Kustomize or custom code that generates Kubernetes manifests for each service based on the generated Git repository structure. The ApplicationSet will ensure consistent application configuration across multiple environments.

RBAC Configuration: Implement RBAC policies in the Kubernetes API server to enforce team-based access controls. Define roles and role bindings within the cluster’s namespace, ensuring that only authorized team members can commit changes to their respective repositories.

CI/CD Pipeline Integration: Integrate this setup with a CI/CD pipeline tool (e.g., Jenkins, GitLab CI) to automate the deployment process. The pipeline will trigger when new services are added or existing ones are updated, ensuring that all stages of development and deployment follow GitOps best practices.

Monitoring and Validation: Set up monitoring tools like Prometheus and AlertManager to track application health and ensure compliance with RBAC policies. Use tools like Argo CD for continuous synchronization between the desired state (Git repositories) and the actual cluster state.

#### Closing Summary
+ Common Mistakes/Follow-Ups: By combining a Git-directory-generator with an ApplicationSet, we can automate the onboarding of new services while maintaining strict RBAC controls. However, common pitfalls include overcomplicating the directory structure, neglecting proper RBAC configuration, and not integrating robust monitoring tools to ensure compliance.

In follow-up discussions, we might explore additional topics such as how to handle edge cases (e.g., service migrations), the impact of different CI/CD strategies, and best practices for maintaining a GitOps pipeline.

**Comparison:**

| Role | Permissions                           | Namespace  |
|------|---------------------------------------|------------|
| Dev  | Read/Write                            | dev        |
| QA   | Read                                  | qa         |
| Prod | Read, but no modifications            | prod       |

**Code:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: new-service-appset
spec:
  generator: {}
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

**Example input / output:**

| Input: New Service Onboarding Request | Output: Generated Git Repository and Kubernetes Manifests |
|---------------------------------------|--------------------------------------------------------|
| User submits a new service request with details. | A new Git repository is created under `services/<SERVICE_NAME>` directory. The `Chart.yaml` and `values.yaml` are populated with default values. |

**Data:**

```json
{
  "service-repo-template": {
    "Chart.yaml": {
      "apiVersion": "v2",
      "name": "<SERVICE_NAME>",
      "version": "0.1.0"
    },
    "values.yaml": {
      // Default values for the service
    }
  }
}
```

**Flow:**

1. New Service OnboardingRequest
2. Git-directory-generator
3. Generate Git Repositories
4. ApplicationSet ManifestGenerator
5. Kubernetes API Server RBACCheck
6. CI/CD Pipeline Triggered
7. Prometheus and AlertManagerMonitoring

