---
type: "practice"
difficulty: "Deep"
---

# A third-party GitHub Action in your migration pipeline was compromised and could exfiltrate your production database credentials. What supply-chain controls (SHA-pinning actions, least-privilege GITHUB_TOKEN, secret scoping) should have been in place, and your response now?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Preventing Secret Leaks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Preventing_Secret_Leaks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a third-party GitHub Action compromises your migration pipeline and risks exfiltrating production database credentials, it's crucial to implement robust supply-chain controls. These controls ensure that even if an action is compromised, the damage remains contained. The key controls include SHA-pinning actions, using least-privilege GITHUB_TOKEN, and secret scoping. Let’s break down each control step by step.

Firstly, SHA-pinning ensures that only trusted versions of a GitHub Action are executed. This involves adding specific commit hashes (SHAs) to your pipeline configuration, so it only runs actions with those exact versions. Secondly, using a least-privilege GITHUB_TOKEN means limiting the scope and permissions of tokens used within your CI/CD pipelines to minimize potential damage if a token is compromised. Lastly, secret scoping helps manage secrets like database credentials by storing them securely and restricting access to only necessary components.

**Comparison:**

```
Control	Description	Example Implementation
SHA-pinning	Ensures actions run only specific, trusted versions.	uses: https://github.com/external/migration-action.git@SHA-TO-PIN
Least-privilege GITHUB_TOKEN	Limits the scope of permissions for tokens.	permissions: { contents: read, id-token: write }
Secret scoping	Manages secrets securely and restricts access to necessary components.	secrets.DB_CREDENTIALS: ${{ secrets.DB_CREDENTIALS }}
```

**Code:**

```yaml
# Example pipeline configuration with SHA-pinning and secret scoping

name: Database Migration

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  # Job to run the migration script
  migrate-database:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    - name: Setup Python (if using a Python action)
      uses: actions/setup-python@v4
      with:
        python-version: 3.9
    - name: Use specific version of the migration action
      id: use-action
      run: echo "::set-output name=action::https://github.com/external/migration-action.git@SHA-TO-PIN"
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    - name: Run migration script
      # Replace with your actual action and secret scoping steps
      uses: ${{ steps.use-action.outputs.action }} # Use the pinned version
      with:
        DB_CREDENTIALS: ${{ secrets.DB_CREDENTIALS }}
```

**Example input / output:**

```text
Input:
- `SHA-TO-PIN` is the specific commit hash of a trusted version of the migration action.
- `secrets.DB_CREDENTIALS` contains an encrypted database credential.

Output:
- Pipeline runs the trusted version of the migration action.
- The pipeline uses the least-privileged `GITHUB_TOKEN`.
- Database credentials are securely stored and accessed via secrets scoping.
```

**Data:**

```json
{
  "secrets": {
    "DB_CREDENTIALS": "encrypted-value",
    "GITHUB_TOKEN": "encrypted-value"
  }
}
```

**Flow:**

1. Yes
2. No
3. GitHub Action
4. Is SHA-pinned?
5. Migrates database
6. Fail pipeline and notify securityteam
7. Database migrated safely
8. Notify developers to updateaction

