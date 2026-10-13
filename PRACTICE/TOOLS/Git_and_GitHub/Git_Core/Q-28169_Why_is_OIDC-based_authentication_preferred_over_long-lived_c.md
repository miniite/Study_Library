---
type: "practice"
difficulty: "Expert"
---

# Why is OIDC-based authentication preferred over long-lived cloud keys in a GitHub Actions deploy job?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ OIDC Federation for Deploys](<../../../../TOOLS/Git_and_GitHub/Git_Core/OIDC_Federation_for_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In software development, secure access to resources like GitHub repositories and other cloud services is crucial. When dealing with automated deployments using GitHub Actions, choosing between OIDC (OpenID Connect) authentication and long-lived cloud keys involves weighing security trade-offs and operational ease. OIDC-based authentication is generally preferred because it provides a more secure and dynamic approach compared to the static nature of long-lived cloud keys.

TL;DR: OIDC-based authentication for GitHub Actions deploy jobs offers better security, flexibility, and reduced management overhead by leveraging short-lived tokens that are automatically managed by the service provider.

#### Triaging the Question

The question is probing your understanding of secure access control mechanisms in CI/CD pipelines. The key aspects to consider include:

Security: How well each method protects against unauthorized access.
Flexibility and Scalability: Ease of use, adaptability to different environments, and ability to handle frequent changes.
Management Overhead: Effort required for provisioning, revoking, and renewing credentials.

#### Assumptions
You understand the basics of GitHub Actions and OIDC.
You are familiar with concepts like SSH keys and service accounts in cloud environments.
Step-by-Step Solution
1. Understanding Long-Lived Cloud Keys

Long-lived cloud keys involve creating an SSH key pair that is stored securely and used to authenticate your actions or services within the cloud environment. While this method can be simpler to set up, it has several drawbacks:

Security Risks: The private key could be leaked if not properly secured.
Management Overhead: Rekeying involves generating new keys, updating configurations, and ensuring that all services are updated with the new public key.
2. Exploring OIDC-Based Authentication

OIDC-based authentication is a more modern approach that leverages short-lived tokens issued by Identity Providers (IdPs) like Azure AD or Okta. This method provides several benefits:

Dynamic Tokens: Tokens are created on-the-fly and have limited lifetimes, reducing the window for potential misuse.
Revocation: If a token is compromised, it can be revoked immediately, limiting damage.
Integration with IdPs: Easier to integrate with existing identity management systems, providing a single sign-on (SSO) experience.
3. Technical Details of OIDC in GitHub Actions

When using OIDC for GitHub Actions:

GitHub App: You create a GitHub App that acts as an Identity Provider.
Token Generation: The GitHub App generates short-lived tokens on behalf of users or services, which are then used to authenticate actions.

Here’s how it works in practice:

User Authentication: A user authenticates using their credentials (username and password).
Token Issuance: The IdP issues a token.
GitHub Action Execution: The action uses the token to access the GitHub API or other services.
4. Practical Example

Let’s consider a simple flow:

YAML
name: Sample Job

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Authenticate with OIDC
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: echo "Authenticated using OIDC token"

      - name: Deploy to cloud
        uses: actions/deploy-to-cloud@v1
        with:
          deployment_key: ${{ secrets.DEPLOY_KEY }}

#### Closing Summary

In summary, OIDC-based authentication is preferred over long-lived cloud keys for GitHub Actions deploy jobs due to its enhanced security features and lower management overhead. Dynamic tokens provide better protection against unauthorized access and facilitate easier revocation and management.

#### Common Mistakes/Follow-Ups
Failing to Rotate Tokens: Ensure that tokens are rotated regularly.
Misconfiguring OIDC Setup: Double-check the setup in IdPs and GitHub Actions for any misconfigurations.
Not Using Environment Variables Properly: Securely store sensitive information like API keys and tokens.

**Code:**

```yaml
name: Sample Job

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Authenticate with OIDC
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: echo "Authenticated using OIDC token"

      - name: Deploy to cloud
        uses: actions/deploy-to-cloud@v1
        with:
          deployment_key: ${{ secrets.DEPLOY_KEY }}
```

