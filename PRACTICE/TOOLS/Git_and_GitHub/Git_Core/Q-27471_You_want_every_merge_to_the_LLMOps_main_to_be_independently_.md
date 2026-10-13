---
type: "practice"
difficulty: "Deep"
---

# You want every merge to the LLMOps main to be independently revertable so on-call can roll back one prompt feature without touching others. What commit and merge discipline enforces this, and how does squash-merge help or hurt?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Merge Strategies](<../../../../TOOLS/Git_and_GitHub/Git_Core/Merge_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that every merge to the LLMOps main branch is independently revertable, we need a robust git workflow that enforces commit and merge discipline. Specifically, this involves using feature branches for individual development tasks, carefully structured commit messages, and utilizing squash-merge judiciously. Squash-merge helps in combining multiple commits into a single one on the target branch, which can simplify the revert process. However, it also has some downsides that need to be managed.

First, let's summarize the key points:

Independent Revertability: We want each feature change to be isolated and easily reversible without affecting other features.
Feature Branches: Each new development task should have its own branch.
Commit Messages: They must be descriptive and follow a consistent format.
Squash-Merge: Helps in maintaining cleaner main branches but can complicate revert operations if not managed well.

Now, let's dive deeper into the details:

**Comparison:**

```
Commit Message Structure	Example
Type(Descriptive Summary)	feat(prompt for feature 123): Add prompt for feature 123
```

**Code:**

```bash
# Example workflow steps
git checkout -b feature-prompt-123 origin/main  # Create a new feature branch from main
# Make changes and commit them
git add .
git commit -m "Add prompt for feature 123"  # Descriptive commit message
```

**Example input / output:**

```text
# Example input and output of squash-merge process
Input:
1. Developer creates `feature-prompt-123` branch.
2. Commits changes to this branch.

Output:
1. Pull request created from `feature-prompt-123` to `main`.
2. Team reviews, approves, and performs a squash-merge into `main`.

Revert:
1. A new branch is created: `revert-feature-prompt-123`.
2. Changes are made to revert the specific commit(s) from `main`.
3. The revert is then merged back into `main` using squash.
```

**Data:**

```json
{
    "commit": {
        "message": "Fix linting issues in the feature-prompt-123 branch",
        "hash": "abc123def456"
    },
    "merge_request": {
        "title": "Add prompt for feature 123",
        "description": "This merge introduces a new prompt for feature 123, ensuring it is independently revertable.",
        "branch_name": "feature-prompt-123"
    }
}
```

**Flow:**

1. Create Feature Branch
2. Develop and Commit Changes
3. Open Pull Request on GitHub
4. Code Review and Approval
5. Squash-Merge to Main
6. Revert Specific Commits ifNecessary
7. Revert Merged Feature UsingSquash Merge

