---
type: "practice"
difficulty: "Deep"
---

# After a squash-merge of a big schema feature, teammates who branched off the pre-squash work keep re-introducing already-applied migrations into their branches. Explain the DAG reason and the exact workflow to reset their branches cleanly onto main.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Reverting Merges](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverting_Merges.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a team member performs a squash-merge of a large schema feature, it can create complications in downstream branches if those branches have already been updated with some or all of the migrations applied during the merge. The issue arises because the branch may not be fully rebased on the main branch after the squash-merge, leading to conflicts and redundant migrations.

To understand why this happens, let's first look at the Git history as a Directed Acyclic Graph (DAG). Each commit in Git is represented by a node, with directed edges indicating the parent-child relationship between them. When you perform a squash-merge, all changes from the feature branch are consolidated into a single commit on main, potentially bypassing some of the intermediate commits that were applied to other branches.

In this scenario, teammates who branched off before the squash-merge might have already merged in some of those migrations directly, making their branches diverge slightly from the updated main. When they rebase or merge back into main, conflicts can occur because the same migration appears both as a direct merge and during the base update.

To resolve this cleanly, we need to reset these branches to align with the current state of main after the squash-merge. The exact workflow involves two steps: rebasing onto main and then reapplying any missing migrations.

Rebase onto Main: This step ensures that all local changes are applied on top of the latest main branch, resolving conflicts where necessary.
Sync Migrations: After rebase, we need to check if any new migrations were introduced in the squash-merge and apply them to ensure consistency with main.

Let's walk through this workflow with a concrete example:

Branches Before Squash-Merge:

Main: A -> B
Feature-Branch: A -> C1 -> C2
Teammate-A: A -> D1 (merges C1)
Teammate-B: A -> E1

After Squash-Merge, Main becomes: A -> B -> Merged-FEATURE (squashed commit)

Teammates with Unmerged Changes:

Teammate-A has A -> D1
Teammate-B has A -> E1

Workflow for Teammate-A:

BASH
# Move to Teammate's branch
git checkout Teammate-A

# Fetch latest main and rebase onto it
git fetch origin main
git rebase origin/main  # This will resolve conflicts if any

# Now, apply the missing migrations from the squash-merge
# Assuming we know which migrations were applied in Merged-FEATURE (e.g., using Git blame or diff)
git merge --no-commit origin/main  # Merge to see what needs rebasing
git rebase --continue  # Continue with the rebase, applying only necessary changes

# Now check if any new migrations need to be applied
git log --since="1 week" origin/main  # List recent commits on main

Repeat the same steps for Teammate-B:

BASH
# Move to Teammate's branch
git checkout Teammate-B

# Fetch latest main and rebase onto it
git fetch origin main
git rebase origin/main  # Resolve conflicts if any

# Now, apply the missing migrations from the squash-merge
git merge --no-commit origin/main  # Merge to see what needs rebasing
git rebase --continue  # Continue with the rebase, applying only necessary changes

# Check for new migrations
git log --since="1 week" origin/main  # List recent commits on main

By following this workflow, teammates can ensure their branches are fully up to date and free from redundant or conflicting migrations.

**Comparison:**

```
Step	Action	Description
1	Checkout branch	Switch to the teammate's branch.
2	Fetch main	Update local tracking of remote main.
3	Rebase onto main	Apply all changes from main as if they were made on the current branch.
4	Merge (no commit)	Temporarily merge in the latest main to see what needs rebasing.
5	Resolve conflicts	Manually resolve any conflicts introduced by the rebase.
6	Continue rebase	Complete the rebase process with only necessary changes.
7	Sync migrations	Ensure all new migrations from main are applied.
```

**Code:**

```

git checkout Teammate-A git fetch origin main git rebase origin/main git merge --no-commit origin/main git rebase --continue

CODE
Copy

```bash
git checkout Teammate-B
git fetch origin main
git rebase origin/main
git merge --no-commit origin/main
git rebase --continue
```

**Example input / output:**

```text
Input (Teammate-A):
git checkout Teammate-A
git fetch origin main
git rebase origin/main

Output:
Resolving conflicts...
Applying necessary changes...

Input (Teammate-A, after resolving conflicts):
git merge --no-commit origin/main

Output:
Auto-merging schema/migration.py
CONFLICT (content): Merge conflict in schema/migration.py
Automatic merge failed; fix conflicts and then commit.

Input (Teammate-A, resolving conflicts):
git rebase --continue

Output:
Applying necessary changes...

Input (Teammate-B):
git checkout Teammate-B
git fetch origin main
git rebase origin/main

Output:
Resolving conflicts...
Applying necessary changes...

Input (Teammate-B, after resolving conflicts):
git merge --no-commit origin/main

Output:
Auto-merging schema/migration.py
CONFLICT (content): Merge conflict in schema/migration.py
Automatic merge failed; fix conflicts and then commit.

Input (Teammate-B, resolving conflicts):
git rebase --continue

Output:
Applying necessary changes...
```

**Data:**

```
Sample branches and commits:
CODE
Copy
Main: A -> B -> Merged-FEATURE
Teammate-A: A -> D1 (merges C1)
Teammate-B: A -> E1
```

**Flow:**

1. Checkout Teammate-A Branch
2. Fetch origin/main
3. Rebase onto origin/main
4. Merge --no-commitorigin/main
5. Resolve conflicts if any
6. Continue rebase with gitrebase --continue
7. Sync migrations from main

