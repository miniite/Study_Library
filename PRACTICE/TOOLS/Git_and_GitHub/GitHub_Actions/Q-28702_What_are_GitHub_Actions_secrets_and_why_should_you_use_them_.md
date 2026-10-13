---
type: "practice"
difficulty: "Basic"
---

# What are GitHub Actions secrets and why should you use them instead of hardcoding values?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Secrets in Actions](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Secrets_in_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

GitHub Actions secrets are environment variables that store sensitive information like API keys, passwords, or other confidential data. Using secrets in GitHub Actions is crucial because it enhances security by keeping your credentials out of your workflow scripts, ensuring they aren't accidentally committed to the repository.

#### Triaging the Question
and Assumptions

The core of this question revolves around understanding the concept of secrets management in GitHub Actions. You should assume that you have a basic knowledge of using environment variables in scripts and workflows. The interviewer is probing your understanding of security best practices, specifically how to protect sensitive information in automation pipelines.

#### First-Principles Solution
What Are Secrets?

Secrets in GitHub Actions are a way to securely store sensitive data like API tokens or passwords. These secrets are encrypted at rest and in transit, which means they are protected from unauthorized access during both storage and transmission.

Why Use Them Instead of Hardcoding Values?
Security: Hardcoded values can be easily seen by anyone who has access to the repository. Secrets ensure that sensitive information is not stored within your scripts or repositories.
Compliance: Many organizations have strict compliance policies regarding data handling, which often mandate using secrets management practices.
Ease of Management: You can manage and update secrets without changing your workflow files, making it easier to maintain and scale your workflows.
Access Control: Secrets can be controlled at the repository level, allowing you to grant access only to those who need it.

#### Example
Usage

To use a secret in a GitHub Action, you first define it using actions-secrets. For example:

YAML
name: My Workflow

on:
  push:
    branches: [ main ]

jobs:
  my-job:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Use a secret
      env:
        API_KEY: ${{ secrets.API_KEY }}
      run: echo "Using API key: $API_KEY"

In this example, the API_KEY is referenced using ${{ secrets.API_KEY }}.

Security Best Practices
Encryption: Secrets are encrypted both at rest and in transit.
Access Control: You can control who has access to these secrets through repository permissions.
Rotation: Regularly rotate your secrets to reduce the risk of exposure.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, GitHub Actions secrets provide a robust way to manage sensitive data securely in your workflows. By using them instead of hardcoding values, you enhance the security and compliance posture of your automation pipelines. Common mistakes include not using secrets at all or misconfiguring their access control.

During an interview, expect follow-ups such as:

How do you manage multiple environments (development vs production)?
What happens if a secret is accidentally committed to the repository?
Can you explain how GitHub Actions integrates with other tools for better security?

**Comparison:**

| Secret Type | Description                       |
|-------------|-----------------------------------|
| API Key     | A token used for API authentication. |
| Password    | An encrypted password             |
| Token       | A unique identifier               |

**Code:**

```yaml
name: My Workflow

on:
  push:
    branches: [ main ]

jobs:
  my-job:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Use a secret
      env:
        API_KEY: ${{ secrets.API_KEY }}
      run: echo "Using API key: $API_KEY"
```

**Example input / output:**

```

Input (GitHub Actions YAML):

YAML
Copy
name: My Workflow

on:
  push:
    branches: [ main ]

jobs:
  my-job:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Use a secret
      env:
        API_KEY: ${{ secrets.API_KEY }}
      run: echo "Using API key: $API_KEY"

Output (Run): Using API key: <secret-value>
```

**Data:**

```yaml
name: My Workflow

on:
  push:
    branches: [ main ]

jobs:
  my-job:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Use a secret
      env:
        API_KEY: ${{ secrets.API_KEY }}
      run: echo "Using API key: $API_KEY"
```

**Flow:**

1. Define Secrets
2. Use Secrets in Workflows
3. Encrypt and Securely Store
4. Maintain Access Control
5. Rotate Secrets Regularly

