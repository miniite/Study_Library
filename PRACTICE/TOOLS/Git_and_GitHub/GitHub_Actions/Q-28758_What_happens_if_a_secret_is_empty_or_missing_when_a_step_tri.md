---
type: "practice"
difficulty: "Intermediate"
---

# What happens if a secret is empty or missing when a step tries to use it, and how would you catch that?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Secrets in Actions](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Secrets_in_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using GitHub Actions, secrets are often used to store sensitive information like API keys, passwords, etc., which need to be handled securely. If a secret is either empty or missing when a step tries to use it, several issues can arise, such as runtime errors, security vulnerabilities, and pipeline failures. To effectively handle these scenarios, we need to understand how secrets are managed in GitHub Actions and implement appropriate error handling mechanisms.

**Code:**

```yaml
# Example GitHub Actions workflow with secret usage
name: Example Workflow

on:
  push:
    branches: [ main ]
  schedule:
    - cron: '0 * * * *'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16'

      - name: Secret Usage Step
        env:
          API_KEY: ${{ secrets.API_KEY }}
        run: |
          if [ -z "${{ env.API_KEY }}" ]; then
            echo "API_KEY is empty or missing"
            exit 1
          fi

          # Example usage of the secret
          echo "Using API Key: ${API_KEY}"
```

**Example input / output:**

```
Input - Secret Usage Step in GitHub Actions

if [ -z "${{ env.API_KEY }}" ]; then echo "API_KEY is empty or missing" exit 1 fi

echo "Using API Key: ${API_KEY}"

CODE
Copy

# Output - Example Scenario

Run if [ -z "${{ env.API_KEY }}" ]; then echo "API_KEY is empty or missing" ; fi using -e bash -c 'if [ -z "$\r${{ env.API_KEY }}\r" ]; then echo "API_KEY is empty or missing"; fi' Using API Key: <secret>

CODE
Copy

# Output - Secret Missing Scenario

Run if [ -z "${{ env.API_KEY }}" ]; then echo "API_KEY is empty or missing" ; fi using -e bash -c 'if [ -z "$\r${{ env.API_KEY }}\r" ]; then echo "API_KEY is empty or missing"; fi' Using API Key: API_KEY is empty or missing
```

**Flow:**

1. Yes
2. No
3. Secret Usage Step
4. Is API_KEY present?
5. Proceed with the step
6. Log an error and exit pipeline

