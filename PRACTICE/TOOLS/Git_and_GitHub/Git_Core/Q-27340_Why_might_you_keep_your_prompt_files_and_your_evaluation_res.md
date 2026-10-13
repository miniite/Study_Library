---
type: "practice"
difficulty: "Intermediate"
---

# Why might you keep your prompt files and your evaluation results moving together in one commit so a reviewer sees both the change and its effect?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Versioning Experiments with Git](<../../../../TOOLS/Git_and_GitHub/Git_Core/Versioning_Experiments_with_Git.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working on AI projects, it's essential to maintain transparency and traceability of changes made during development. Keeping prompt files and their corresponding evaluation results together in one commit serves this purpose well. This practice ensures that a reviewer can see the exact modifications made to the prompts and immediately observe how these changes impact the model’s performance or output. Essentially, it provides a complete snapshot of the change process.

To understand why this is beneficial, let's break down what the question is probing and explore the solution step-by-step from first principles.

**Code:**

```
MERMAID
Copy
graph TB
    A[Commit Prompt Files] --> B[Store Changes]
    C[Commit Evaluation Results] --> B
    D[Bug Fixes or Improvements] --> B
    E[Complete Snapshot] --> B
    F[Traceability and Transparency] --> E
```

**Example input / output:**

```text
No input-output examples are necessary here, as the concept is more about best practices than a specific workflow.
```

