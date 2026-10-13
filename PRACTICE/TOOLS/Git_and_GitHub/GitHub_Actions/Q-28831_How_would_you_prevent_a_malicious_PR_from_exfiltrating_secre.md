---
type: "practice"
difficulty: "Expert"
---

# How would you prevent a malicious PR from exfiltrating secrets via a modified workflow file, considering pull_request vs pull_request_target?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pinning Third-party Actions](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Pinning_Third-party_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To prevent a malicious Pull Request (PR) from exfiltrating secrets via a modified GitHub Actions workflow file, we need to ensure that changes to sensitive configurations are detected and blocked before they can be merged into our main codebase. The difference between pull_request and pull_request_target workflows lies in their triggers: pull_request runs on any PR opened or updated within the repository, while pull_request_target only runs when a PR is pushed to the remote branch.

#### Triaging the Question

The core of this question is understanding how to secure sensitive information against unauthorized access and exfiltration. Specifically, we need to:

Detect if a PR contains changes to sensitive files or configurations.
Block the merge if such changes are detected.
Automate this process using GitHub Actions.

#### Assumptions
We assume that secrets are stored in environment variables, files, or external vaults.
We have access to the necessary permissions and tools within our CI/CD pipeline.
We are familiar with YAML syntax for defining GitHub Actions workflows.
Step-by-Step Solution

Environment Variables: Define sensitive information as environment variables in a .env file. Ensure this file is not committed to the repository by setting skip_commit: true in your .gitignore.

Action Workflow Structure:

YAML
name: Secret Detection and Protection

on:
  pull_request_target:

jobs:
  detect-secrets:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Setup Node.js (if applicable)
        if: startsWithrunner.os, 'Linux'
        run: sudo apt-get update && sudo apt-get install -y nodejs

      - name: Install Dependencies
        run: npm ci

      - name: Detect Secrets
        id: detect-secrets
        uses: github/codeql-action/analyze@v2
        with:
          languages: javascript # or other relevant language

      - name: Fail if Secrets Detected
        if: steps.detect-secrets.outputs.has-detected-secrets == 'true'
        run: echo "Secrets detected! Aborting merge."
        exit-code: 1

Customizing the Workflow:

Use github/codeql-action to detect secrets in code.
Integrate with other tools like snyk, secrets-scanner, or custom scripts if needed.

Additional Security Measures:

Code Reviews: Ensure PRs are reviewed by multiple team members before merging.
Secret Management Tools: Use tools like AWS Secrets Manager, Azure Key Vault, or HashiCorp Vault to manage secrets securely.
Access Control: Limit who can modify sensitive files and workflows.

#### Closing Summary

By leveraging GitHub Actions with the pull_request_target event, we can automatically detect and prevent malicious PRs from exfiltrating secrets. The key steps involve setting up environment variables, configuring actions for secret detection, and integrating additional security measures. This approach ensures that any changes to sensitive configurations are thoroughly vetted before they can be merged into our main codebase.

#### Common Mistakes
Overlooking Environment Variable Security: Failing to ignore or protect .env files in the repository.
Lack of Continuous Monitoring: Not setting up continuous monitoring and alerts for secret detection.
Insufficient Code Reviews: Relying solely on automation without human oversight.
Inadequate Access Controls: Allowing too many users to modify sensitive configurations.

