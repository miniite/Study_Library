---
type: "practice"
difficulty: "Deep"
---

# What is a GitHub deploy key and when would you use one instead of a personal token?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Token Permissions](<../../../../TOOLS/Git_and_GitHub/Git_Core/Token_Permissions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

GitHub deploy keys are a security feature that allows repository access without using a personal access token (PAT). Instead, they provide limited, predefined permissions to specific repositories or branches. You would use a GitHub deploy key when you need an automated service like a CI/CD pipeline to push code changes to a repository in the name of the repository itself rather than relying on a user's credentials.

#### Triage

The question is probing your understanding of secure methods for managing access control and automation in GitHub repositories. By explaining what a GitHub deploy key is, when it should be used, and comparing it with personal tokens, you can demonstrate your grasp of security best practices.

#### First-Principles Solution
What Is a GitHub Deploy Key?

A GitHub deploy key is an RSA public-private key pair generated for the purpose of authenticating the Git client to a repository. The private key is stored securely on the CI/CD server or another automated service, and the public key is added to the repository's deployment keys settings in the GitHub UI.

When to Use a Deploy Key

You would use a deploy key when:

Automated Services: You need an automated process (like a Jenkins job) to push changes to a repository. This ensures that sensitive credentials are not exposed.
CI/CD Pipelines: For integrating with continuous integration and deployment workflows where the build server needs to commit or tag code.
Security Best Practices: To minimize access risks, you can limit the permissions of deploy keys to specific repositories or branches.
Comparison with Personal Tokens
Personal Access Token (PAT): A long-lived token that provides fine-grained access control. It is stored in environments and used for API calls or direct repository interactions.
Deploy Key: Provides more restricted, machine-to-machine authentication. Typically, it grants push permissions to a specific branch and does not expose sensitive information like passwords.

#### Closing Summary

In summary, GitHub deploy keys are an effective security mechanism for automated services that need to interact with repositories without exposing personal tokens. They offer a balance between convenience and security by limiting the scope of access to only what is necessary.

#### Common Mistakes/Follow-Ups
Incorrect Permissions: Ensure that deploy keys have the minimum permissions required, as overly permissive keys can increase security risks.
Key Rotation: Regularly rotate your deploy keys to minimize exposure in case they are compromised.
Environment Variables: Store sensitive information like repository URLs and branches as environment variables rather than hardcoding them.
Self-Check

This solution covers the core concepts and practical usage of GitHub deploy keys, ensuring both correctness and completeness for a deep understanding level question.

**Comparison:**

```
Feature	GitHub Deploy Key	Personal Token (PAT)
Scope	Limited to repo/branch	Wide ranging access
Usage	CI/CD, automation services	API calls, direct Git operations
Security	Better security as it is machine-to-machine	Higher risk if compromised as it can be used in multiple contexts
```

**Code:**

```
MARKDOWN
Copy
# Example Repository Settings
1. Generate an RSA key pair.
2. Add the public key to repository settings under "Settings" -> "Deploy Keys".
3. Use the private key in your CI/CD pipeline configuration.
```

**Example input / output:**

```text
Input:
- Repository URL: https://github.com/user/repo.git
- Branch: main

Output:
- Deploy Key is used to push changes to the 'main' branch of the repository without exposing a PAT.
```

**Data:**

```yaml
- Public Key: ssh-rsa AAAAB3...
- Private Key: kept secret and stored securely
```

**Flow:**

1. Yes
2. No
3. Deploy Key
4. Is the service automated?
5. Use Deploy Key
6. Use Personal Token (PAT)

