---
type: "practice"
difficulty: "Intermediate"
---

# What is RBAC and how does it relate to controlling who can deploy in GitOps?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Multi-tenant GitOps](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Repo_and_Environment_Layout/Multi-tenant_GitOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

RBAC stands for Role-Based Access Control, a security model that allows system administrators to define roles and grant permissions based on those roles. In the context of GitOps, RBAC is crucial for ensuring that only authorized individuals or teams can push changes to production environments. This prevents unauthorized deployments and ensures compliance with organizational policies.

#### Triaging the Question

The question asks you to explain RBAC and its relation to controlling who can deploy in GitOps. The interviewer is likely probing your understanding of:

What RBAC is: Its definition, components, and how it works.
How RBAC fits into GitOps: Why RBAC is essential for GitOps workflows and how it integrates with the broader GitOps strategy.

#### First-Principles Solution

Let's start by defining RBAC in a GitOps context:

RBAC Definition:

In GitOps, RBAC ensures that only individuals or teams with specific roles can perform certain actions, such as deploying changes to production.
Roles are predefined sets of permissions (e.g., read-only access, write access, admin privileges).

How RBAC Relates to GitOps:

GitOps is a practice that aims to make infrastructure as code and maintain consistency between development and operations teams by using Git for version control.
RBAC integrates with GitOps by defining who can push changes (e.g., manifests or configuration files) into the Git repository, which then triggers automated deployment processes.
Step-by-Step Solution

Defining Roles:

Administrators: Can create and manage roles.
Developers: Can make code changes but not necessarily deploy them.
Deployers: Can apply changes to production environments based on predefined roles.

Assigning Permissions:

Assign specific permissions (e.g., read, write) to each role.
Ensure that only authorized individuals can perform actions like merging pull requests or pushing code to the Git repository.

Automating Deployment:

Use CI/CD pipelines to automatically deploy changes from the Git repository to infrastructure as code (IaC).
Ensure that RBAC is implemented in these pipelines, so only users with the correct roles can trigger deployments.

Monitoring and Auditing:

Implement logging and auditing to track who deployed what and when.
Use RBAC to restrict access based on these logs for enhanced security.

#### Closing Summary

RBAC is a critical component of GitOps that ensures secure and controlled deployment processes. By defining roles and assigning permissions, you can prevent unauthorized deployments and maintain compliance with organizational policies. This approach not only enhances security but also streamlines the development and operations workflow in a GitOps environment.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to mention that RBAC should be implemented at multiple levels (e.g., repository-level, branch-level).
Mistake 2: Not discussing how RBAC integrates with CI/CD pipelines.
Mistake 3: Ignoring the importance of monitoring and auditing in maintaining security.

You might get follow-up questions like:

How do you implement RBAC in a GitOps environment?
What are some best practices for defining roles and permissions?
Can you give an example of how RBAC affects the deployment process?

**Comparison:**

```
Role	Permissions
Admin	All actions
Developer	Read-only access
Deployer	Write access
User	Roles
john.doe	developer
jane.doe	deployer
```

**Code:**

```yaml
# Example YAML file defining roles and permissions

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]

---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: developer-binding
subjects:
- kind: User
  name: john.doe
  apiGroup: ""
roleRef:
  kind: Role
  name: developer-role
  apiGroup: rbac.authorization.k8s.io
```

**Example input / output:**

```text
# Example input for deploying a pod with RBAC

Input:
- User: john.doe
- Action: Create Pod

Output:
- Result: Forbidden (john.doe has only read permissions)

Dry-run:
1. Check if the user is part of the 'developer' role.
2. Allow reading pods but not creating them.
```

**Data:**

```json
{
  "users": [
    {
      "name": "john.doe",
      "roles": ["developer"]
    },
    {
      "name": "jane.doe",
      "roles": ["deployer"]
    }
  ],
  "permissions": [
    {"role": "developer", "resources": ["pods"], "verbs": ["get", "list"]},
    {"role": "deployer", "resources": ["pods"], "verbs": ["create", "patch"]}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Authenticates
6. Is the user a Developer?
7. Read access granted
8. Is the user a Deployer?
9. Create and Patch accessgranted
10. Forbidden

