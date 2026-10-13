---
type: "practice"
difficulty: "Intermediate"
---

# What information can you get from the github context, for example github.actor or github.ref?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Workflow Fundamentals](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Workflow_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with GitHub Actions, understanding the github context is crucial as it provides essential metadata about the current workflow execution environment. The github.actor, github.ref, and other related keys give you insights into who initiated the action, which branch or tag was triggered, and more. Let's break down what each of these pieces of information means and how they can be useful in your workflows.

**Code:**

```yaml
# Example GitHub Actions workflow
on:
  push:
    branches: [ main ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Display GitHub context variables
        run: |
          echo "Triggered by: ${{ github.actor }}"
          echo "Branch: ${{ github.ref }}"
```

**Example input / output:**

```

Here’s a simple example of how these values might be used in an input-output scenario:

Input (GitHub Event):

github.actor: "octocat"
github.ref: "refs/heads/main"

Output (Action Step Output):

CODE
Copy
Triggered by: octocat
Branch: refs/heads/main
```

