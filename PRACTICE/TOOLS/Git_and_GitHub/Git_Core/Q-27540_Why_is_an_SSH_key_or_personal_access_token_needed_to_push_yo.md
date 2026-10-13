---
type: "practice"
difficulty: "Basic"
---

# Why is an SSH key or personal access token needed to push your observability project to GitHub instead of a password?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Token Permissions](<../../../../TOOLS/Git_and_GitHub/Git_Core/Token_Permissions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why an SSH key or personal access token (PAT) is necessary when pushing an observability project to GitHub, we need to consider the security and performance implications of using passwords versus these more secure alternatives.

#### Triage

The core of this question revolves around security best practices in version control systems like Git. Passwords are not recommended due to their potential vulnerabilities and inconvenience during frequent interactions with repositories. SSH keys or PATs provide a safer and more efficient way to authenticate users, ensuring that sensitive credentials aren't stored insecurely.

#### First-Principles Solution
Security: Storing passwords in plain text is inherently risky because they can be easily accessed by malicious actors.
Efficiency: SSH keys and PATs simplify the authentication process, reducing the need for repeated password entry during frequent operations like pushing code.
Scalability: In scenarios with multiple contributors or automated pipelines, managing many passwords becomes cumbersome.
Detailed Explanation:
Security Risks
Storing Passwords: When you use a password to authenticate via HTTPS (the default method), your password is transmitted in plain text over the network and stored temporarily on GitHub’s servers. This exposes your credentials to potential eavesdroppers.
Credential Management: Managing passwords can be challenging, especially when multiple team members are involved or when dealing with automated tools that require frequent authentication.
Efficiency Gains
SSH Keys: An SSH key is a pair of cryptographic keys (public and private) used for secure communication over unsecured networks. When you configure your local machine to use an SSH key, the public key is added to your GitHub account, allowing it to verify your identity without requiring a password.
PLAINTEXT
Local Machine -> Public Key (GitHub Server) -> Private Key (Local Machine)
Personal Access Tokens: A PAT acts similarly to a password but provides more granular access control and security features. You can generate a PAT in GitHub settings, which is then used as an alternative to your password.
PLAINTEXT
Local Machine -> Personal Access Token (GitHub API) -> User Account
Scalability Considerations
Multiple Contributors: In a large organization with many developers, managing passwords for each user can become cumbersome. SSH keys and PATs streamline the process by centralizing authentication.
Automated Pipelines: When setting up continuous integration/continuous deployment (CI/CD) pipelines, you don't want to hard-code credentials into scripts or repositories. Using SSH keys or PATs ensures that your automated processes are secure and scalable.

#### Closing Summary

In summary, using an SSH key or a personal access token is crucial for securing interactions with GitHub, especially when dealing with frequent operations like pushing code or automating workflows. These methods offer better security, efficiency, and scalability compared to passwords.

#### Common Mistakes/Follow-Ups
Mistake: Using plain text passwords in Git URLs (e.g., https://username:password@github.com/repo).

#### Follow-Up
Ensure that all sensitive information is managed securely using SSH keys or PATs.
Mistake: Hardcoding credentials in scripts or repositories.

#### Follow-Up
Use environment variables, secret management tools (like Hashicorp Vault), or secure vaults to manage secrets.

**Comparison:**

```
Authentication Method	Security Level	Efficiency	Scalability
Passwords	Low	Moderate	Poor
SSH Keys	High	High	Good
Personal Access Tokens	Medium-High	High	Excellent
```

**Code:**

```plaintext
# Example of setting up an SSH key
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa

# Example of adding a PAT in GitHub settings
```

**Example input / output:**

```text
# Example input and output for adding an SSH key to the list of authorized keys on GitHub

User adds their public SSH key to their GitHub account.
GitHub responds with a confirmation message.

# Example input and output for using a PAT in Git operations

User runs `git push` with a URL that includes a PAT.
Git performs the push operation securely without requiring user interaction for authentication.
```

**Data:**

```yaml
# Sample GitHub API request using a PAT
GET /repos/{owner}/{repo}/contents/{path}
Authorization: token {personal_access_token}
```

**Flow:**

1. Local Machine
2. Generate SSH Key
3. Add Public Key to GitHub
4. Use PAT in Git Operations
5. Perform Secure Push

