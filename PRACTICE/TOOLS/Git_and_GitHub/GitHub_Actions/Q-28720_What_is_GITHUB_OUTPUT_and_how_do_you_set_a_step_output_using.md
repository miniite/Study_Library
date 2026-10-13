---
type: "practice"
difficulty: "Deep"
---

# What is GITHUB_OUTPUT and how do you set a step output using it?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Workflow Fundamentals](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Workflow_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

GitHub Actions are automation tools that help you automate your development, testing, and deployment processes in Git repositories. The GITHUB_OUTPUT file is a powerful feature of GitHub Actions that allows you to set environment variables that can be accessed by subsequent steps within the same job or workflow.

To understand how GITHUB_OUTPUT works, let's break it down step by step.

**Code:**

```yaml
# Example YAML snippet for a GitHub Action
steps:
  - name: Set an output value using GITHUB_OUTPUT
    run: echo "hello=world" >> $GITHUB_OUTPUT

  - name: Use the output from the previous step
    run: echo ${{ steps.set-output.outputs.hello }}
```

