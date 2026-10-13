---
type: "practice"
difficulty: "Intermediate"
---

# How does GitHub secret scanning with push protection try to stop a token from ever reaching the remote?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Token Permissions](<../../../../TOOLS/Git_and_GitHub/Git_Core/Token_Permissions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Secret scanning and push protection in GitHub are designed to secure your repositories by identifying and blocking sensitive information like API tokens before they can be pushed to the remote repository. This mechanism is crucial for preventing unauthorized access and ensuring data security.

To understand how secret scanning with push protection works, we need to break down the core concepts and process from first principles. GitHub's secret scanning service scans the commit history of your repository for known secrets such as API tokens. Push protection then ensures that any pushes containing these secrets are automatically rejected by the repository’s hooks or webhooks.

Let's dive into the detailed steps:

Secret Scanning: When a push is made to a repository, GitHub uses secret scanning to analyze the commit content and history for known patterns of sensitive information. These include API tokens, passwords, private keys, etc.
Push Protection Configuration: You configure push protection by setting up rules in your repository's settings. These rules define what constitutes a "secret" and how they should be handled if detected during a push.
Detection and Blocking: If secret scanning detects any secrets that match the defined patterns, it triggers an automatic rejection of the push request. The push is blocked before it can reach the remote repository.

Key Assumptions:

You have properly set up secret scanning in your GitHub settings.
Your repository has relevant rules configured for push protection.
The API token or sensitive data you are concerned about follows common patterns that GitHub’s scanners can detect (e.g., length, character distribution).

From First Principles:

Initial Commit: Consider a scenario where a developer accidentally commits an API token to the repository.
Secret Scanning Trigger: When this commit is pushed, secret scanning runs on it.
Detection: If the API token matches known patterns or is part of a custom rule set, it is flagged as a potential security risk.
Push Protection Activation: The push request is intercepted by GitHub’s push protection rules before reaching the remote repository.
Outcome: The push is blocked, and an alert is sent to the configured security team.

#### Closing Summary
Secret scanning with push protection in GitHub provides a robust layer of security by actively preventing sensitive information from being pushed to your repositories. It works by analyzing commits for patterns matching known secrets and leveraging push protection rules to block pushes containing such secrets before they reach the remote repository. This helps prevent unauthorized access and ensures that critical data remains secure.

#### Common Mistakes/Follow-Ups

Not configuring secret scanning or push protection properly.
Committing sensitive information without proper safeguards.
Ignoring alerts from secret scanning, which can lead to accidental exposure of secrets.
Overlooking edge cases where custom rules might not cover all possible scenarios.

**Comparison:**

```
Step	Action	Result
1	Commit code	Codebase update
2	Secret scanning	Token detection
3	Push protection	Push rejection
```

**Code:**

```python
# Example of a simple script to simulate pushing code with a token
def push_code_with_token(token):
    import requests
    url = "https://api.github.com/user/repos"
    headers = {"Authorization": f"token {token}"}
    response = requests.post(url, json={"name": "test-repo"}, headers=headers)
    return response.status_code

# Simulating a scenario where the token is detected as sensitive
def simulate_secret_scanning(token):
    if "API_KEY" in token:
        print("Secret detected!")
        return 403  # Forbidden - Push blocked
    else:
        print("Push successful.")
        return 201  # Created - Push allowed

status_code = push_code_with_token("API_KEY")
print(f"Status code: {status_code}")
```

**Example input / output:**

```text
Input:
- Developer pushes code containing an API token.
Output:
- Secret scanning triggers and detects the API token.
- Push protection rules block the push due to the detected secret.
```

**Data:**

```json
{
  "repository": {
    "name": "test-repo",
    "settings": {
      "secretScanningEnabled": true,
      "pushProtectionRules": [
        {
          "pattern": "API_KEY",
          "action": "BLOCK"
        }
      ]
    }
  },
  "commit": {
    "message": "Add API key to the codebase.",
    "content": "import os\napi_key = 'API_KEY'"
  }
}
```

**Flow:**

1. Developer pushes code
2. Secret scanning triggers
3. Detection of API token
4. Push protection rules activated
5. Push blocked, alert sent
6. Security team notified
7. Repository protected fromunauthorized access

