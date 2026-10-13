---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between the env context and the secrets context?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Secrets in Actions](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Secrets_in_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In GitHub Actions, the env context and the secrets context are both used to manage sensitive information like API keys or database passwords, but they serve different purposes and have distinct usage scenarios.

The TL;DR is that env is for storing non-sensitive environment variables, whereas secrets is specifically designed for handling sensitive data securely. This difference impacts how these contexts should be used in your GitHub Actions workflows to ensure both usability and security.

To triage the question:

The interviewer is probing your understanding of GitHub Actions' security features.
You need to explain the differences between two context types (env and secrets) used for storing variables.
It’s important to highlight that while both can be used to store environment variables, secrets are intended for sensitive data.

Let's start from first principles. Environment variables in GitHub Actions can be stored using the env key in your workflow file. These variables are defined as plain text and can be easily seen by anyone with access to the repository or the logs generated during a run. For example:

YAML
name: My Workflow

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      API_KEY: ${{ secrets.API_KEY }}
      DATABASE_URL: http://localhost:5432

Here, DATABASE_URL is stored in plain text, while API_KEY is stored as a secret.

On the other hand, sensitive data such as API keys, passwords, and tokens should be stored using the secrets context. This ensures that these values are encrypted at rest and only decrypted for use during the execution of the workflow. Secrets provide an additional layer of security by preventing them from being accidentally exposed in logs or commit history. For example:

YAML
name: My Workflow

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      API_KEY: ${{ secrets.API_KEY }}
      DATABASE_URL: http://localhost:5432

The key difference is that secrets are stored in an encrypted format, whereas env variables can be plain text. Additionally, secrets have a more secure handling mechanism and are subject to additional validation checks.

Now let's reason step-by-step from first principles:

Understanding the Contexts:

env: Used for storing non-sensitive environment variables that need to be accessible during the workflow execution.
secrets: Specifically designed for sensitive data like API keys, passwords, and tokens.

Usage Scenarios:

Use env when you have general configuration settings or credentials that are not considered sensitive.
Use secrets when storing sensitive information to ensure secure handling and encryption at rest.

Security Implications:

Storing sensitive data in plain text with env can lead to accidental exposure, especially if someone gains unauthorized access to the repository.
Using secrets ensures that even if an attacker gains access to the logs or commit history, they won't be able to see the actual secrets.

Best Practices:

Always prefer using secrets for sensitive data.
Avoid storing sensitive information in plain text with env.
Regularly review and update your secrets, especially if you use external services or APIs that require frequent key rotation.

In summary, while both env and secrets can be used to store environment variables in GitHub Actions, the key difference lies in their intended purpose. Use env for non-sensitive data and secrets for sensitive information to ensure better security practices.

**Code:**

```yaml
name: My Workflow

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      API_KEY: ${{ secrets.API_KEY }}
      DATABASE_URL: http://localhost:5432
```

