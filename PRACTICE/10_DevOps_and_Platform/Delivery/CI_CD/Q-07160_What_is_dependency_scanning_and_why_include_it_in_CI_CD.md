---
type: "practice"
difficulty: "Intermediate"
---

# What is dependency scanning and why include it in CI/CD?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ CI vs Continuous Delivery vs Deployment](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_vs_Continuous_Delivery_vs_Deployment.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Dependency scanning, a critical part of CI/CD pipelines, ensures that all project dependencies are up-to-date and secure. By integrating this process into your continuous integration and deployment workflows, you can proactively identify vulnerabilities, manage dependency versions, and ensure compliance with security policies.

The primary goal is to prevent issues such as outdated libraries or potential security risks before they cause problems in production. In this context, CI/CD pipelines are like a conveyor belt that processes code changes; dependency scanning fits into the process at key stages to catch these issues early.

First, let's understand what dependency scanning involves:

Identifying Dependencies: The pipeline scans for all dependencies used by the project.
Analyzing Dependencies: It evaluates these dependencies against a database of known vulnerabilities and versions.
Reporting Issues: Any identified vulnerabilities or outdated packages are reported back to the development team.

Now, let's break down why dependency scanning is crucial in CI/CD:

Why Include Dependency Scanning in CI/CD?
Security Vulnerabilities: Outdated libraries can introduce security risks like SQL injection, cross-site scripting (XSS), and other vulnerabilities.
Version Management: Ensuring that all dependencies are up-to-date helps maintain compatibility across the project and prevents breaking changes introduced by older versions.
Compliance: Many organizations have strict policies regarding dependency usage to avoid legal or security breaches.
Step-by-Step Solution
1. Identify Dependencies

The first step is identifying all the dependencies in your project. This can be done using tools like npm-check, pip-review for Python, or equivalent tools specific to other languages.

BASH
# Example: Using npm-check
npm-check
2. Analyze Dependencies

Next, these identified dependencies are analyzed against a database of known vulnerabilities and version issues. Tools like Snyk, Dependabot, and Black Duck are commonly used for this purpose.

JSON
// Example output from Snyk
{
  "name": "example-app",
  "dependencies": {
    "lodash": {
      "version": "4.17.21",
      "vulnerabilities": [
        {
          "id": "SNYK-JS-LODASH-965806",
          "title": "Prototype Pollution in Lodash and Underscore",
          "severity": "high"
        }
      ]
    },
    "axios": {
      "version": "0.21.1",
      "vulnerabilities": []
    }
  }
}
3. Reporting Issues

The pipeline should generate a report highlighting any issues found, which can then be addressed by the development team.

BASH
# Example: Snyk command to run and output findings
snyk test --org=your-organization-name

#### Implementation
in CI/CD

To integrate dependency scanning into your CI/CD pipeline, you need to add a step that runs these tools as part of the build process. This could be done using YAML configuration files for popular CI/CD platforms like GitHub Actions or GitLab.

YAML
# Example: GitHub Actions workflow
name: Dependency Scanning

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  dependency-scanning:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '14'

    - name: Install dependencies
      run: npm ci

    - name: Run Snyk Dependency Scanning
      run: snyk test --fail-on-vuln=high

#### Closing Summary
and Common Mistakes

Dependency scanning is essential for maintaining secure, up-to-date projects in CI/CD pipelines. However, common mistakes include skipping the step altogether or not addressing issues identified by the scans. It's crucial to have a clear process for reviewing and resolving any vulnerabilities reported.

Always keep your dependencies updated and regularly scan them to ensure security and compliance.

**Comparison:**

```
Dependency	Version	Vulnerabilities
lodash	4.17.21	SNYK-JS-LODASH-965806 (High)
axios	0.21.1	None
```

**Code:**

```yaml
# Example GitHub Actions workflow for dependency scanning
name: Dependency Scanning

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  dependency-scanning:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '14'

    - name: Install dependencies
      run: npm ci

    - name: Run Snyk Dependency Scanning
      run: snyk test --fail-on-vuln=high
```

**Example input / output:**

```text
# Example output from Snyk command to run and output findings
Snyk detected vulnerabilities in your dependencies.
- lodash version 4.17.21 has a high-severity vulnerability (SNYK-JS-LODASH-965806).
- axios is up-to-date with no known vulnerabilities.

Run `snyk test --fail-on-vuln=high` to integrate this into your CI/CD pipeline.
```

**Data:**

```json
{
  "name": "example-app",
  "dependencies": {
    "lodash": {
      "version": "4.17.21",
      "vulnerabilities": [
        {
          "id": "SNYK-JS-LODASH-965806",
          "title": "Prototype Pollution in Lodash and Underscore",
          "severity": "high"
        }
      ]
    },
    "axios": {
      "version": "0.21.1",
      "vulnerabilities": []
    }
  }
}
```

**Flow:**

1. Code Check-in
2. Run Dependency Scanning
3. Identify Vulnerabilities
4. Generate Report
5. Review and Fix Issues
6. Commit Changes
7. Push to Repository

