---
type: "practice"
difficulty: "Intermediate"
---

# How do fine-grained personal access tokens differ from classic PATs in terms of scope and safety?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Token Permissions](<../../../../TOOLS/Git_and_GitHub/Git_Core/Token_Permissions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Fine-grained personal access tokens (PATs) are a more secure and granular alternative to classic PATs, designed to limit access based on specific permissions rather than the broad roles or scopes used by traditional PATs. In this context, we’ll compare the scope and safety aspects of both types.

#### Triage

The question asks about the differences between fine-grained personal access tokens (FG-PATs) and classic PATs in terms of their scope and safety. The key areas to explore are:

Scope: How FG-PATs vs. classic PATs define and limit permissions.
Safety: Measures taken to ensure secure usage, such as token management, expiration policies, and revocation.

#### Assumptions
We assume both types of tokens are used in a Git/GitHub environment for authentication and authorization purposes.
Classic PATs are the traditional method where users obtain a single-token with broad permissions, often covering multiple repositories or operations.
FG-PATs introduce more granular permission sets to tailor access based on specific needs.

#### First-Principles Solution
Scope

Classic PATs:

Typically provide all-encompassing permissions within an organization or repository, e.g., "read/write" for a user across multiple projects.
Often granted by administrators to users with broader roles (e.g., maintainer, collaborator).

FG-PATs:

Offer more granular control over permissions at the level of individual repositories, branches, and specific actions like pull requests, deployments, or commit messages.
Permissions can be scoped down to fine levels such as "read", "write" for a particular branch, or "approve reviews" only on certain PRs.
Safety

Classic PATs:

Limited in terms of granular control over permissions.
If compromised, the impact is broad, potentially affecting multiple repositories and operations within an organization.

FG-PATs:

Provide enhanced security through fine-grained permission sets, reducing the attack surface if a token is stolen or misused.
Easier to manage and revoke specific permissions without impacting other parts of the repository or organization.

#### Implementation
Example
Yes
Granular permission checks
Access granted
Insufficient permissions
No
FG-PAT
Is token used?
Check permissions
Verify access level
Perform operation
Deny access
Token revoked or expired
Revoke access

#### Closing Summary

FG-PATs offer a more secure and granular approach to managing permissions compared to classic PATs. By providing fine-grained control, FG-PATs minimize the impact of token compromise and make it easier to manage and revoke specific permissions without affecting other parts of the repository or organization.

#### Common Mistakes/Follow-Ups
Mistake: Overusing FG-PATs for all operations can complicate management and auditing.

#### Follow-Up
Consider using FG-PATs selectively in critical areas, such as CI/CD pipelines, to ensure security without overcomplicating everyday development workflows.

**Code:**

```
MARKDOWN
Copy
// Example of a simple function that generates an FG-PAT with specific scopes
function generateFGPAT(repoName, permissions) {
    return `token=${generateRandomToken()}&scopes=${permissions.join(',')}`;
}
```

**Example input / output:**

```

Given the input:

repoName: example-repo
permissions: [read:branches, write:pulls, admin:deploys]

Output:

JSON
Copy
{
  "token": "generated-token",
  "scopes": ["read:branches", "write:pulls", "admin:deploys"]
}
```

**Data:**

```json
{
  "repoName": "example-repo",
  "permissions": ["read:branches", "write:pulls", "admin:deploys"]
}
```

