---
type: "practice"
difficulty: "Deep"
---

# A GitHub Actions concurrency or environment block in YAML is not gating deploys as expected. How do you read the YAML precedence between job-level and workflow-level concurrency?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Refactoring Helm Values](<../../../../TOOLS/YAML/YAML_Core/Refactoring_Helm_Values.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When troubleshooting a GitHub Actions workflow that isn’t behaving as expected due to concurrency settings, it’s crucial to understand how concurrency and environment blocks interact at both the job level and the workflow level in YAML configurations. This involves parsing the precedence rules and ensuring your understanding aligns with best practices.

In this context, you’re looking for a clear breakdown of how these blocks influence job execution and the conditions under which concurrency is enforced. The key idea here is that concurrency settings at both levels can override or complement each other, depending on their configuration. Let’s dive into the details from first principles.

**Code:**

```yaml
# Example workflow with concurrency and environment blocks
name: My Workflow

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    concurrency:
      group: ${{ github.sha }}
      cancel-in-progress: true
    steps:
      - name: Checkout code
        run: echo "Checking out code"

  test:
    runs-on: macos-latest
    environment: qa
    concurrency:
      group: ${{ github.sha }}
      cancel-in-progress: true
    steps:
      - name: Run tests
        run: echo "Running tests in QA environment"
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. GitHub Action Workflow
6. Is Concurrency Set at JobLevel?
7. Check Job-level ConcurrencySettings
8. Is Concurrency Set at WorkflowLevel?
9. Check Workflow-levelConcurrency Settings
10. No Concurrency SettingApplied
11. Job is Gated by ConcurrencyGroup and Cancel-in-ProgressRule
12. Workflow is Gated byConcurrency Group andCancel-in-Progress Rule
13. Job Executes After PrecedenceRules are Met
14. Job Executes After PrecedenceRules are Met
15. No Jobs Execute Due to NoConcurrency Setting Applied

