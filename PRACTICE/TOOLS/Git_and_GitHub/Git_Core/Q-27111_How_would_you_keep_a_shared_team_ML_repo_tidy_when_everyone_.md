---
type: "practice"
difficulty: "Intermediate"
---

# How would you keep a shared team ML repo tidy when everyone tends to commit their own experiment notebooks?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Sharing Projects on GitHub](<../../../../TOOLS/Git_and_GitHub/Git_Core/Sharing_Projects_on_GitHub.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To keep a shared team ML repo tidy, especially with multiple members committing individual experiment notebooks, we need to establish clear practices and workflows that promote organization, version control, and reproducibility. This involves setting up proper commit messages, branch management, and possibly using GitHub Actions for automated tasks like testing and documentation.

First, let's summarize the key points: The goal is to ensure that everyone can work independently while maintaining a clean, organized repository. We need to balance individual freedom with team standards.

**Code:**

```bash
# Example of a good commit message format
git commit -m "Refactor Experiment Notebook A for improved accuracy and added comments"
```

**Example input / output:**

```text
Example commit messages:

- `Experiment notebook B updated with new hyperparameters`
- `Fix typo in README.md`
- `Add experiment C to track model performance over time`

These messages are clear, concise, and describe the changes made.
```

