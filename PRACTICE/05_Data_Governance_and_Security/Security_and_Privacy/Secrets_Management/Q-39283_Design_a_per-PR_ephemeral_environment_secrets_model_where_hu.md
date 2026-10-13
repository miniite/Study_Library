---
type: "practice"
difficulty: "Deep"
---

# Design a per-PR ephemeral environment secrets model where hundreds of preview envs spin up and down daily. How do you avoid leaking prod secrets into previews and clean up dynamic leases automatically?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shared Secret Rotation](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Shared_Secret_Rotation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an effective per-PR (Pull Request) ephemeral environment secrets model, we need to ensure that secrets are securely managed, isolated from production environments, and dynamically cleaned up after their use. The core idea is to leverage a secrets management solution like HashiCorp Vault, paired with automation tools such as Jenkins or GitOps workflows, to manage the lifecycle of these secrets.

#### Triage

The question asks for a design that avoids leaking production (prod) secrets into preview environments and automatically cleans up dynamic leases when the associated environment is no longer needed. The key points are:

Isolation: Ensure that secrets used in previews do not leak into prod.
Dynamic Leases: Automatically manage secret lifetimes to prevent indefinite retention.

#### Assumptions
We use HashiCorp Vault for secrets management.
The preview environments spin up and down based on PRs.
Each PR corresponds to a unique environment, and the environment name includes the PR ID.
Secrets are versioned and can be rotated dynamically.
Automation tools like Jenkins or GitOps manage the provisioning and de-provisioning of these environments.

#### Solution
Secret Versioning and Rotation:
Store secrets in Vault with versioning enabled. This ensures that each secret has a unique version number, making it easy to track and rotate them.
Environment Names and Tags:
Use environment names like preview-<PR_ID> or <BRANCH_NAME>-preview. This helps in identifying the environment's purpose quickly.
Dynamic Leases:
Configure Vault to use dynamic leases, which automatically expire after a certain period of inactivity (e.g., 24 hours).
Secret Binding and Unbinding:
Bind secrets to specific environments when they are provisioned. When the environment is torn down, unbind the secrets.
Role-Based Access Control (RBAC):
Implement RBAC within Vault to ensure that only authorized users can access secrets in preview environments.
Step-by-Step Solution
Environment Creation:
Trigger a CI/CD pipeline when a PR is opened.
The pipeline provisions the environment and requests secret bindings from Vault using dynamic tokens or a predefined role for this purpose.
Secret Binding:
Once the environment is provisioned, Vault binds the required secrets to it. This can be done using token policies that are specific to the environment name.
Dynamic Leases:
Set up dynamic leases in Vault with appropriate TTL (Time-To-Live). For example, a lease might expire after 24 hours of inactivity.
When the PR is merged or closed, the CI/CD pipeline should automatically unbind and delete the secrets from the environment.

#### Closing Summary

By leveraging HashiCorp Vault's dynamic leases, role-based access control, and versioning capabilities, we can ensure that secrets used in preview environments are isolated, secure, and automatically cleaned up. This approach minimizes the risk of production secrets leaking into previews and ensures efficient management of secret lifecycles.

#### Common Mistakes/Follow-Ups
Mistake: Failing to implement RBAC properly, leading to unauthorized access.

#### Follow-Up
How do you handle situations where multiple environments need the same set of secrets?
Mistake: Not configuring dynamic leases correctly, resulting in indefinite retention of secrets.

#### Follow-Up
What happens if a PR environment is not deleted due to a failure?

**Comparison:**

```
Environment	Status	Secrets Bound
preview-1234	Active	db-credentials, api-key
```

**Code:**

```yaml
# Vault Token Policy for Preview Environments
path "secret/preview-1234/*" {
  capabilities = ["read"]
}

# Jenkinsfile snippet for provisioning an environment and requesting secrets
pipeline {
    stages {
        stage('Provision Environment') {
            steps {
                script {
                    def envName = "preview-${params.PR_ID}"
                    // Provision the environment here...
                    // Request secrets from Vault using a dynamic token or role
                    sh 'vault read -token=<dynamic-token> secret/preview-1234/*'
                }
            }
        }
    }
}
```

**Example input / output:**

```text
Input:
- PR ID: 1234

Output:
Environment "preview-1234" is provisioned, and the following secrets are bound:
- secret/preview-1234/db-credentials
- secret/preview-1234/api-key

After 24 hours of inactivity or when PR #1234 is closed, these secrets will be unbound and deleted from "preview-1234".
```

**Data:**

```yaml
# Vault Secret Configuration
path "secret/production" {
  capabilities = ["read"]
}

path "secret/preview-*" {
  capabilities = ["read"]
}
```

**Flow:**

1. Yes
2. PR Opened
3. Provision Env?
4. Environment Provisioned
5. Request Secrets from Vault
6. Grant Tokens & Bind Secrets
7. Wait for PR Closure orInactivity
8. Revoke Tokens & UnbindSecrets
9. Tear Down Env

