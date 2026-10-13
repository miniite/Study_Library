---
type: "practice"
difficulty: "Intermediate"
---

# What is the continue-on-error setting and when would a fresher use it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Reusable_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In GitHub Actions, continue-on-error is a setting that allows workflows to continue executing even if a job or step fails, which can be crucial for ensuring comprehensive test coverage and avoiding unnecessary halts in your CI/CD pipeline.

#### Triage

The question asks you to explain the continue-on-error setting and when it would be useful. The key points are:

What is continue-on-error?
When should a DevOps fresher use this setting?

#### First Principles Solution
Understanding continue-on-error: This flag allows GitHub Actions to proceed with subsequent jobs or steps in the workflow, even if one of the previous jobs or steps fails. By default, workflows halt on failure, but enabling continue-on-error ensures that all defined actions are executed regardless of intermediate errors.
Use Cases for Freshers:
Comprehensive Test Coverage: In a fresh DevOps environment, you might have various types of tests (unit tests, integration tests, etc.) spread across different jobs in your workflow. Using continue-on-error ensures that even if one test fails, other tests still run, providing more detailed insights into the state of your application.
Avoiding Unnecessary Halts: Freshers might not be aware that a single failure can halt the entire workflow, leading to missed opportunities for additional testing or notifications. Enabling continue-on-error helps avoid such interruptions and ensures that all necessary actions are completed.

#### Closing Summary

In summary, continue-on-error is a powerful feature in GitHub Actions that allows workflows to continue execution even if some steps fail, ensuring comprehensive test coverage and avoiding unnecessary halts. It’s particularly useful for freshers who might not be aware of the implications of failure on the overall workflow.

#### Common Mistakes/Follow-Ups
Mistake: Forgetting to enable continue-on-error in workflows that have complex dependencies.

#### Follow-Up
Discussing how to handle specific failures using conditions or error-handling strategies, such as running only certain jobs based on previous success/failure states.

**Code:**

```yaml
on:
  push:
    branches: [ main ]
jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Node.js environment
        uses: actions/setup-node@v1
        with:
          node-version: '14'

      - name: Install dependencies
        run: npm install

      - name: Run unit tests
        continue-on-error: true
        run: npm test

      - name: Run integration tests
        run: npm run test:integration
```

