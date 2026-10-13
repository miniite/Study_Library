---
type: "practice"
difficulty: "Intermediate"
---

# How would you use branches so a data-loader change and a prompt change to your RAG app can be reverted independently later?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Git Worktrees](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Worktrees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working on an AI application, it's crucial to manage changes independently without affecting each other. For instance, in a Retrieval-Augmented Generation (RAG) application, you might want to update data loaders and prompts separately, ensuring that if issues arise with one change, the other can be reverted independently. Git branches are the perfect tool for this purpose.

#### Triage

The question is asking how to use Git branches effectively so that changes to a data loader and a prompt in an RAG application can be managed independently. Specifically, we need to ensure that both sets of changes are tracked separately, allowing us to revert them independently if needed.

#### Assumptions
You have basic knowledge of Git commands.
The project is hosted on GitHub for version control.
Both the data loader and prompt changes are significant enough to warrant separate branches.

#### Solution
from First Principles

Identify the Changes: Determine which parts of your codebase require independent tracking.

In this case, it's the data loader (e.g., data_loader.py) and a prompt-related component (e.g., prompt_engine.py).

Create Separate Branches:

Create a branch for the data loader changes: git checkout -b update-data-loader.
Create a separate branch for the prompt changes: git checkout -b update-prompt.

Make Changes and Commit: Apply your changes in each respective branch.

BASH
# For the data loader change
git checkout update-data-loader
# Make your data loader modifications

# Stage, commit, and push to GitHub
git add .
git commit -m "Update data loader"
git push origin update-data-loader

# Repeat the same process for the prompt changes

Test Changes Independently:

Before merging, test both branches independently to ensure they work as expected.
BASH
# Pull the latest updates from remote and switch back to main branch (or a development branch)
git checkout main
git pull origin main

# Test the data loader changes
git checkout update-data-loader
python -m pytest tests/data_loader_tests.py

# Test the prompt changes
git checkout update-prompt
python -m pytest tests/prompt_engine_tests.py

Merge Changes Independently:

Once both branches are tested and working, merge them back into the main branch.
BASH
# Merge data loader changes first
git checkout main
git pull origin main

git merge --no-ff update-data-loader
git push origin main

# Then merge prompt changes
git checkout main
git pull origin main

git merge --no-ff update-prompt
git push origin main

Revert Changes if Needed:

If issues arise with one change, you can revert it independently.
BASH
# Reverting data loader changes
git checkout main
git revert HEAD

# Reverting prompt changes
git checkout main
git revert HEAD

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By using separate branches, you can manage changes to the data loader and prompts independently. This approach ensures that both sets of changes are tracked and tested separately, making it easier to revert them if necessary.

#### Common Mistakes
Not creating a branch for each set of changes.
Merging all changes into one branch without proper testing.
Failing to push branches to GitHub for backup.
Common Follow-Ups
How do you handle conflicts when merging branches?
What if there's a conflict between the data loader and prompt changes during merge?
Can you walk through an example of pushing these branches to GitHub?

