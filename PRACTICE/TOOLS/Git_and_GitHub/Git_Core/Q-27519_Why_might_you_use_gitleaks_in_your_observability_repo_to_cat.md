---
type: "practice"
difficulty: "Intermediate"
---

# Why might you use gitleaks in your observability repo to catch an accidentally committed Grafana or PagerDuty API token?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Preventing Secret Leaks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Preventing_Secret_Leaks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with a monitoring and observability repository, ensuring that sensitive information like API tokens is not accidentally committed is crucial. gitleaks is a tool designed to detect secrets in your codebase by scanning commit messages and file contents. In the context of an AIOps team using Grafana or PagerDuty for monitoring and alerting, it’s essential to prevent such tokens from being exposed. Here’s why you might use gitleaks:

Security: Sensitive information like API keys can be accidentally committed if not properly managed.
Compliance: Many organizations have strict security policies that require the protection of sensitive data.
Risk Mitigation: Exposure of tokens could lead to unauthorized access or data breaches.

Now, let’s break down how gitleaks works and why it is beneficial in this scenario:

#### Triage
What the Question Is Asking: The question asks about using gitleaks to detect accidentally committed Grafana or PagerDuty API tokens.
What the Interviewer Is Probing: The interviewer wants to understand your understanding of sensitive information management, security practices, and familiarity with tools like gitleaks.

#### Assumptions
We are dealing with an observability repository containing configuration files and scripts for monitoring and alerting systems like Grafana and PagerDuty.
Developers might accidentally commit API tokens in these files or commit messages.
Step-by-Step Solution

Understanding gitleaks:

gitleaks is a command-line tool that can be integrated into your CI/CD pipeline to perform static analysis on your codebase and commit history.
It looks for common secrets such as API keys, tokens, passwords, and more.

Setting Up gitleaks:

Install gitleaks using package managers like npm, yarn, or pip.
BASH
npm install -g gitleaks

Configuring gitleaks for Specific Secrets:

You can create a configuration file to specify which secrets you want to detect.
YAML
# .gitleaks.yml
secrets:
  - pattern: YOUR_GRAFANA_API_TOKEN_PATTERN
    name: Grafana API Token
  - pattern: YOUR_PAGERDUTY_API_TOKEN_PATTERN
    name: PagerDuty API Token

Running gitleaks in CI/CD:

Integrate gitleaks into your CI pipeline to run it automatically on each commit.
YAML
# .github/workflows/check-secrets.yml
jobs:
  check-secrets:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Install gitleaks
        run: npm install -g gitleaks
      - name: Run gitleaks
        run: gitleaks --config .gitleaks.yml

Handling False Positives:

gitleaks might sometimes generate false positives, especially if the tokens are encrypted or used in a way that doesn’t match common patterns.
You can configure exceptions or adjust the regex patterns to minimize false positives.

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: Using gitleaks ensures that sensitive information like API tokens is not accidentally committed, reducing security risks and ensuring compliance with organization policies.

#### Common Mistakes
Not integrating gitleaks into the CI/CD pipeline.
Ignoring false positives without proper configuration or validation.
Failing to update the secret patterns as new tokens are added.

#### Follow-Ups
How do you manage secrets in your observability repository?
Can you describe a scenario where gitleaks might have prevented a security incident?

**Comparison:**

```
Type	Pattern	Description
Token	YOUR_GRAFANA_API_TOKEN_PATTERN	A pattern for detecting Grafana API tokens
Token	YOUR_PAGERDUTY_API_TOKEN_PATTERN	A pattern for detecting PagerDuty API tokens
```

**Code:**

```bash
# Install gitleaks
npm install -g gitleaks

# Example .gitleaks.yml configuration
secrets:
  - pattern: YOUR_GRAFANA_API_TOKEN_PATTERN
    name: Grafana API Token
  - pattern: YOUR_PAGERDUTY_API_TOKEN_PATTERN
    name: PagerDuty API Token

# CI/CD pipeline example in GitHub Actions
jobs:
  check-secrets:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Install gitleaks
        run: npm install -g gitleaks
      - name: Run gitleaks
        run: gitleaks --config .gitleaks.yml
```

**Example input / output:**

```text
# Example output from gitleaks
Checking secrets in repository...
- [WARNING] Potential Grafana API Token found at path/to/file.js
- [INFO] No PagerDuty API Token detected.
```

**Data:**

```yaml
# Example .gitleaks.yml
secrets:
  - pattern: YOUR_GRAFANA_API_TOKEN_PATTERN
    name: Grafana API Token
  - pattern: YOUR_PAGERDUTY_API_TOKEN_PATTERN
    name: PagerDuty API Token
```

**Flow:**

1. Checkout code
2. Install gitleaks
3. RUN gitleaks --config.gitleaks.yml
4. Check for warnings and infomessages

