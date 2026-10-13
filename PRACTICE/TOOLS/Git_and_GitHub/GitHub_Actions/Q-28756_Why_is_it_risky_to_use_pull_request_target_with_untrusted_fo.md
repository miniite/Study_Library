---
type: "practice"
difficulty: "Deep"
---

# Why is it risky to use pull_request_target with untrusted forks?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pinning Third-party Actions](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Pinning_Third-party_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To begin, let's quickly summarize why using pull_request_target with untrusted forks can be risky in GitHub Actions. TL;DR: Using pull_request_target involves fetching and merging code from external repositories, which could expose your environment to malicious or poorly maintained code.

Now, let's dive into the specifics of what this question is probing and our assumptions. The core idea here is about security risks associated with integrating untrusted forks in a continuous integration/continuous deployment (CI/CD) pipeline using GitHub Actions.

#### Triage
What’s Being Probed
Security Risks: Understanding the potential for malicious code to be introduced into your repository.
Best Practices: Knowing how to safely manage external dependencies and ensure that pull requests from untrusted sources are handled securely.
GitHub Action Workflow: Familiarity with GitHub Actions syntax, particularly pull_request_target, and its implications.

#### Assumptions
You have a basic understanding of GitHub Actions and CI/CD pipelines.
Your environment uses pull_request_target to merge changes from forks into your main branch for testing purposes.
First Principles: The Full Solution

Understanding pull_request_target: This step allows you to specify that the target branch is not just a main or default branch but any pull request from an external fork. Essentially, it merges the specified pull request into the target branch in your repository.

Security Implications:

Malicious Code: Untrusted forks could contain malicious code or vulnerabilities that could compromise your production environment if merged directly.
Code Quality: Poorly maintained repositories might introduce bugs or performance issues, impacting the quality of your project.

#### Implementation
Details:

YAML
# Example GitHub Actions workflow
on:
  pull_request_target:
    types: [labeled]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    if: github.event.action == 'labeled' && github.event.label.name == 'for-testing'
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Go
        if: startsWith/github.ref, 'refs/pull/'
        # Additional setup steps if needed

      - name: Run tests
        run: go test ./...

Mitigation Strategies:

Code Reviews: Ensure that all code changes are thoroughly reviewed before merging.
Static Analysis Tools: Use tools like SonarQube or Bandit to scan for security vulnerabilities.
Sanitize Inputs: Clean and sanitize any inputs from external sources to prevent injection attacks.
Isolated Environments: Run tests in isolated environments (e.g., Docker containers) that do not have access to sensitive data.

#### Closing Summary

In summary, using pull_request_target with untrusted forks poses significant security risks as it allows the merging of potentially malicious or poorly maintained code into your main branch. To mitigate these risks, you should implement a combination of rigorous code reviews, static analysis tools, and isolated test environments.

#### Common Mistakes/Follow-Ups
Not Understanding Risks: Failing to recognize that pull_request_target can introduce external code directly.
Over-relying on Pull Requests: Assuming all pull requests are safe without proper vetting.
Lack of Isolation: Running tests in environments where sensitive data might be exposed.

**Code:**

```yaml
# Example GitHub Actions workflow
 on:
   pull_request_target:
     types: [labeled]

 jobs:
   build-and-test:
     runs-on: ubuntu-latest
     if: github.event.action == 'labeled' && github.event.label.name == 'for-testing'
     steps:
       - name: Checkout repository
         uses: actions/checkout@v2

       - name: Set up Go
         if: startsWith/github.ref, 'refs/pull/'
         # Additional setup steps if needed

       - name: Run tests
         run: go test ./...
```

