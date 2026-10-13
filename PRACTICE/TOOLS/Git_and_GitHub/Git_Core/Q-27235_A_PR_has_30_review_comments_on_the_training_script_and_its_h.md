---
type: "practice"
difficulty: "Intermediate"
---

# A PR has 30 review comments on the training script and its history is a mess of 'fix review' commits. Clean it up with interactive rebase and --autosquash while preserving reviewers' ability to see what changed since their last look.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Interactive Rebase](<../../../../TOOLS/Git_and_GitHub/Git_Core/Interactive_Rebase.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To clean up a messy Git history, particularly after 30 review comments on a training script, we can use interactive rebase and --autosquash in combination. This approach allows us to streamline the commit history while preserving reviewers' context of changes since their last look.

#### Triage

The question is essentially asking for a way to clean up a repository's history by reordering and squashing commits without losing relevant information. The main goal is to reduce noise in the commit log, making it easier to follow the development process while still allowing reviewers to understand the individual changes they've reviewed.

We need to assume that:

The PR has 30 review comments spread across multiple commits.
There are multiple authors and their respective commits.
Commits have a mix of fix, refactor, and new feature types.
Step-by-Step Solution
Identify the Branch: Identify which branch contains the messy history (e.g., feature/new-training-script).
Interactive Rebase: Use git rebase -i to interactively reorder and edit commits.
Automatically Squash Commits: Use --autosquash with interactive rebase to automatically squash related commits.
Preserve Context for Reviewers: Ensure that each reviewer can still see the individual changes they've reviewed by maintaining appropriate commit messages.

#### Implementation

Start Interactive Rebase:

SH
git checkout feature/new-training-script
git rebase -i HEAD~30  # Adjust the number of commits based on your history

Reorder and Squash Commits: In the interactive rebase editor, you will see a list of commits. You can reorder them as needed to group similar changes together.

Automatically Squash Commits: To squash commits using --autosquash, first set up aliases or use a script for consistency:

SH
git config alias.autosquash '!f() { git reset --soft "$1" && git commit -c "$1"; }; f'

Apply Autosquash:

SH
git rebase -i --autosquash HEAD~30  # This will automatically squash and amend commits as needed

Rebase with Context: After reordering and squashing, ensure each commit message is clear and describes the changes accurately. Use git log to verify:

SH
git log --oneline

Push Changes: Push the rebased history back to GitHub ensuring reviewers can still see individual changes using:

SH
git push --force-with-lease

Note: Force pushing might overwrite remote branches, so ensure all collaborators are aware.

#### Closing Summary
and Common Mistakes

By following these steps, we have streamlined the commit history while preserving the context of each reviewer's contributions. This approach ensures a cleaner repository for future reference and review.

#### Common mistakes
to avoid:

Forgetting to use --autosquash in interactive rebase.
Overwriting remote branches without informing collaborators.
Losing the ability to trace individual changes by squashing too aggressively.

