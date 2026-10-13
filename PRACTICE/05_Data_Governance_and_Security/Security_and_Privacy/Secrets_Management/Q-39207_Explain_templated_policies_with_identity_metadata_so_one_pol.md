---
type: "practice"
difficulty: "Expert"
---

# Explain templated policies with identity metadata so one policy grants each user only their own path.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Vault Agent and Auto-auth](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Vault_Agent_and_Auto-auth.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a DevOps environment, managing secrets securely is crucial, especially when multiple users need access to different sets of credentials or sensitive data. Templated policies with identity metadata are an advanced feature in tools like HashiCorp Vault that allow for fine-grained control over secret access based on the user’s identity. In this explanation, I'll walk through how templated policies can be designed to grant each user only their own path, ensuring a secure and scalable secrets management strategy.

#### Triage
Understanding the Question

The core of this question is about implementing an Access Control List (ACL) that leverages Vault's templating capabilities combined with identity metadata. The goal is to ensure that:

Each user can access only their specific paths in Vault.
The policy is dynamic, adapting to changes in the user’s identity without manual intervention.

#### First-Principles Solution
Step 1: Understand Identity Metadata

Vault's identity system allows us to attach metadata to users and services. This metadata could include roles, groups, or unique identifiers that can be used in policies.

Step 2: Define Templated Policies

We'll create a policy template that uses the user’s metadata to construct path-based access rules. The key idea is to use Vault's template function in conjunction with path prefix.

Step 3: Implement Policy Logic

The logic will involve:

Extracting the user’s unique identifier.

#### Cons
tructing a dynamic path based on this identifier.
Granting or revoking access based on the constructed path.
Step 4: Apply Policies to Users

We’ll use Vault’s ACL system to apply these templated policies to users dynamically, ensuring that each user only has access to their own paths.

Detailed Solution

#### Example
Policy Template
HCL
path "secrets/data/user/${.Metadata.username}/*" {
    capabilities = ["read"]
}

path "secrets/keys/user/${.Metadata.username}" {
    capabilities = ["wrap-read", "unwrap"]
}

Here, ${.Metadata.username} is a template variable that will be replaced with the user’s actual username.

Step 1: Create an Identity

First, we create an identity in Vault and attach metadata to it:

HCL
vault write auth/<backend>/user/login name=user123
Step 2: Apply Templated Policy

Next, we apply our templated policy using the dynamic path constructed from the user’s metadata:

HCL
vault policy write user-templates policies=path "secrets/data/user/${.Metadata.username}/*", path "secrets/keys/user/${.Metadata.username}"
Step 3: Map Identity to Policy

Finally, we map this identity to the templated policy:

HCL
vault write auth/<backend>/user/mapping username=user123 policies=user-templates

#### Closing Summary
and Common Mistakes

By leveraging templating in Vault with dynamic path construction based on user metadata, we can create a highly scalable secrets management system where each user has access only to their specific paths. This approach ensures that:

The policy is flexible and adaptable.
Security is enhanced by limiting exposure of sensitive data.

#### Common mistakes
include not properly defining the metadata extraction logic or not correctly mapping identities to policies. It’s also crucial to regularly review and update policies as users are added, removed, or have their roles changed.

**Comparison:**

```
User	Policies Applied
user123	user-templates
```

**Code:**

```hcl
# Example Vault policy template
path "secrets/data/user/${.Metadata.username}/*" {
    capabilities = ["read"]
}

path "secrets/keys/user/${.Metadata.username}" {
    capabilities = ["wrap-read", "unwrap"]
}
```

**Example input / output:**

```
Input: Create an identity with metadata

vault write auth/userpass/user/user123 password=secretpassword

Output: Success message indicating the user has been created

{ "lease_id": "", "lease_duration": 0, "renewable": false, "data": { "username": "user123", "password": "secretpassword" } }

CODE
Copy

```hcl
# Apply templated policy to user mapping
vault write auth/userpass/mapping/user123 policies=user-templates

# Output: Success message indicating the policy has been applied
{
  "policies": [
    "user-templates"
  ]
}
```

**Data:**

```json
{
  "metadata": {
    "username": "user123"
  }
}
```

**Flow:**

1. Create User Identity
2. Attach Metadata?
3. Map User to Policy
4. Grant Access Based onDynamic Path
5. Revoke or Update as Needed

