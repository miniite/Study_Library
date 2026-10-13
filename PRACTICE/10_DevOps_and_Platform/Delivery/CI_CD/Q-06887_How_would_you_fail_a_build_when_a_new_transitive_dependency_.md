---
type: "practice"
difficulty: "Deep"
---

# How would you fail a build when a new transitive dependency introduces a disallowed license, and how do you avoid false positives blocking developers on every dependency bump?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Validating Pipeline Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Validating_Pipeline_Changes/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively manage CI/CD pipelines, ensuring that no disallowed licenses are introduced by new dependencies is crucial. The challenge here lies in failing a build when a transitive dependency contains a restricted license while minimizing false positives to ensure developers can make regular, productive updates.

In essence, the question revolves around how to implement and enforce policy enforcement within CI/CD pipelines to detect and prevent the inclusion of disallowed licenses, and how to handle situations where a developer's bump in dependencies might inadvertently trigger unnecessary build failures. This requires a nuanced understanding of dependency management, license compliance policies, and effective integration with CI/CD tools.

#### Triage
What is Being Probed

The interviewer is probing your knowledge of:

Dependency Management: Understanding how dependencies can be managed and analyzed within the CI/CD pipeline.
License Compliance Policies: Knowing how to define and enforce rules around acceptable licenses, such as avoiding proprietary or closed-source licenses.
Tooling and Automation: Familiarity with tools like dependency scanners, license checkers, and integration with CI/CD systems.
Error Handling and False Positives: Ability to design a system that accurately identifies issues without overly restrictive policies.

#### Assumptions
You are using a modern package manager (e.g., npm, pip) for dependencies.
The team has defined a set of allowed licenses through a policy or configuration file.
There is an existing CI/CD pipeline in place.
License data can be accessed and checked against these policies.
Step-by-Step Solution
1. Define the Policy

First, establish a clear definition of which licenses are disallowed. This could be done by creating a YAML file that lists all disallowed licenses:

YAML
# licenses.yml
disallowed_licenses:
  - 'proprietary'
  - 'closed-source'
2. Integrate a Dependency Scanning Tool

Integrate a tool like Snyk, WhiteSource, or OWASP Dependency-Check into the CI/CD pipeline to scan for these disallowed licenses in transitive dependencies.

SHELL
# Example command using Snyk CLI
snyk test --file=package.json
3. Configure Your CI/CD Pipeline

Use a CI/CD tool like Jenkins, GitHub Actions, or GitLab CI to trigger the dependency scan as part of the build process. For instance, in a GitHub Actions workflow:

YAML
# .github/workflows/ci.yaml
name: CI

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Install dependencies
      run: npm install
    - name: Snyk test
      run: snyk test --file=package.json
    - name: Fail the build if a disallowed license is found
      if: failure()
      run: echo "Build failed due to a disallowed license."
4. Handle False Positives

To avoid false positives, it’s essential to:

Regularly update the dependency scanning tool and policy definitions.
Use whitelists as well as blacklists for licenses.
Implement a manual review process for complex or ambiguous cases.
5. Notify and Provide Guidance

When a build is failed due to a disallowed license, provide detailed feedback to the developer about which specific license was detected, why it’s problematic, and how they can resolve this issue (e.g., by using an alternative dependency).

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, failing a build due to disallowed licenses involves defining clear policies, integrating robust scanning tools, configuring CI/CD pipelines correctly, managing false positives, and providing actionable guidance. Key areas where developers might falter include over-restrictive policies leading to unnecessary build failures or insufficiently detailed error messages that don’t provide enough context for resolution.

#### Common mistakes

Overlooking transitive dependencies.
Failing to update license checks regularly.
Not having a clear process for resolving false positives.
Lack of communication and support for developers when builds fail due to licensing issues.

#### Follow-ups
might include discussions on how to balance strict compliance with flexibility, exploring alternatives like open-source licenses, or discussing the integration of machine learning in detecting new or obscure disallowed licenses.

**Comparison:**

```
Step	Description
1	Define policy for disallowed licenses.
2	Integrate a dependency scanning tool.
3	Configure CI/CD to run scans automatically.
4	Handle and mitigate false positives.
5	Provide detailed feedback to developers.
```

**Code:**

```yaml
# Example GitHub Actions workflow snippet
name: CI

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Install dependencies
      run: npm install
    - name: Snyk test
      run: snyk test --file=package.json
    - name: Fail the build if a disallowed license is found
      if: failure()
      run: echo "Build failed due to a disallowed license."
```

**Example input / output:**

```text
Input: A push or pull request that includes new dependencies, some of which contain a disallowed license.

Output: The CI/CD pipeline fails the build and generates an error message indicating which specific dependency introduced a disallowed license.
```

**Data:**

```yaml
# licenses.yml
disallowed_licenses:
  - 'proprietary'
  - 'closed-source'
```

**Flow:**

1. Yes
2. No
3. Trigger CI
4. Install Dependencies
5. Snyk test --file=package.json
6. License check fails?
7. Fail build, notify developer
8. Publish changes

