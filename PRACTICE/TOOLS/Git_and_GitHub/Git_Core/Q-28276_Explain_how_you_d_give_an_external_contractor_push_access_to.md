---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd give an external contractor push access to exactly one repo and one branch without granting broad org access, comparing deploy keys, fine-grained PATs, and outside-collaborator roles.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ OIDC Federation for Deploys](<../../../../TOOLS/Git_and_GitHub/Git_Core/OIDC_Federation_for_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When giving an external contractor push access to a specific repository and branch on GitHub while minimizing their broader access within the organization, you have several options such as deploy keys, fine-grained Personal Access Tokens (PATs), or outside-collaborator roles. Each method has its trade-offs in terms of security, ease of management, and flexibility.

Firstly, let's consider what each option entails:

Deploy Keys: These are SSH public-private key pairs that can be used to authenticate a Git push or pull without the need for user credentials. They are typically stored on servers that need to interact with your repository.
Fine-Grained Personal Access Tokens (PATs): PATs allow you to grant specific permissions to external users, such as read and write access to particular repositories and branches. You can manage these tokens via the GitHub API or the web interface.
Outside Collaborators: This is a role that can be assigned to an external user, providing them with push access to multiple repositories within the organization without needing to add them as full members.

Now, let's explore each option in detail:

Deploy Keys

Deploy keys are useful when you need automated systems (like CI/CD pipelines) to interact with your repository. Here’s a step-by-step process:

Generate a SSH Key Pair: Use ssh-keygen on the server that needs push access.
Add Public Key to GitHub:
BASH
ssh -T git@github.com
Assign Repository Permissions: Ensure the public key is added under Settings > Deploy keys of the specific repository.

While deploy keys are secure and only allow automated systems to push, they don’t offer the flexibility for human users or the ability to revoke access easily without administrative intervention.

Fine-Grained Personal Access Tokens (PATs)

Fine-grained PATs provide a more flexible way to manage external contributors. You can create a token with specific permissions:

Generate a PAT on GitHub:
BASH
gh auth login
Set Permissions for the PAT: Use gh api or the GitHub UI to set up fine-grained permissions.
Configure Git to Use the PAT:
BASH
git config --global user.name "ContractorName"
git config --global user.email contractor@example.com
git config credential.helper "store --file=.git/credentials"
echo "https://<PAT>:x-oauth-basic@github.com" > .git/credentials

Fine-grained PATs offer better security and manageability since you can revoke or modify the token without altering repository settings. However, it requires more setup for the contractor to configure their Git client.

Outside Collaborators

Outside collaborators are a simpler way to give push access but still provide greater flexibility than deploy keys:

Add User as an Outside Collaborator: Go to Settings > Members in GitHub and add the user.
Set Repository Permissions: In the repository settings, set up read and write permissions for that specific branch.

This method is straightforward and easy to manage through the GitHub UI but still allows the contractor broad access within the organization if they are added as outside collaborators.

Choosing the Right Approach

For an external contractor who needs push access to a single repo and one branch, fine-grained PATs offer the best balance of security and flexibility. You can generate a token with just the necessary permissions and ensure it is used only for that purpose. This way, you avoid giving them broader access within your organization.

#### Conclusion

In summary, using fine-grained Personal Access Tokens (PATs) provides the most secure and granular control over external contributions while minimizing organizational risk. It ensures that contractors have limited but effective access to specific repositories and branches without needing full member status in the organization.

**Code:**

```bash
# Example of generating a PAT
gh auth login
```

