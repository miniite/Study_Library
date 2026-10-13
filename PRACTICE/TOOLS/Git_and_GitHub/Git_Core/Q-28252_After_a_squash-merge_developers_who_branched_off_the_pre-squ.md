---
type: "practice"
difficulty: "Deep"
---

# After a squash-merge, developers who branched off the pre-squash feature keep re-introducing the 'already merged' commits. Explain why and give the workflow to reset their branches cleanly.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Merge Strategies](<../../../../TOOLS/Git_and_GitHub/Git_Core/Merge_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you perform a squash-merge in Git, all of your branch's changes are combined into one commit on the target branch. This means that any commits on your feature branch that were merged with a squash-merge are now represented as a single commit on the main branch. Developers who branched off this pre-squash feature might accidentally reintroduce these 'already merged' commits because their local branches still contain those individual commits, and they may not be aware that they should only have one commit representing all of the changes.

To reset their branches cleanly after a squash-merge, developers need to rebase their local branch onto the main branch. This involves fetching the latest updates from the main branch, rebasing their feature branch on top of these updates, and then force-pushing the updated branch back to GitHub. Let's walk through this process step-by-step.

