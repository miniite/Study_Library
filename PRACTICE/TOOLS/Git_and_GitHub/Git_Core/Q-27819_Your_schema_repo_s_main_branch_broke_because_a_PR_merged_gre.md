---
type: "practice"
difficulty: "Deep"
---

# Your schema repo's main branch broke because a PR merged green against a stale base, missing a conflicting migration merged minutes earlier. Explain what a merge queue plus require-up-to-date solves that individual required checks cannot, and how you'd tune batch size.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Concurrency Groups and Merge Queues](<../../../../TOOLS/Git_and_GitHub/Git_Core/Concurrency_Groups_and_Merge_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with database schemas in a distributed environment like Git/GitHub, it’s crucial to ensure that merges don’t break the codebase, especially when dealing with migrations that alter table structures or add columns. The scenario you described—where a PR merged green against a stale base, missing earlier conflicting migrations—highlights a common problem of manual or individual checks failing to catch all issues.

A merge queue plus require-up-to-date solves this by introducing a systematic approach to ensure that only up-to-date branches can be merged into the main branch. This system works by maintaining a queue where each PR must wait until its base branch is fully caught up with newer migrations before it can be reviewed and potentially merged. This ensures that no conflicting changes are missed, making the merge process more reliable.

To understand this in depth, let's break down how a merge queue plus require-up-to-date works:

Queue System: Each PR is placed into a queue based on its dependency on base migrations.
Up-to-Date Check: Before a PR can be merged, it must wait until all required migrations on the base branch have been applied and merged. This ensures that any changes in the base are fully integrated before the new changes are introduced.
Batch Size Tuning: The batch size is crucial to balance between efficiency and thoroughness. A larger batch size means fewer merge operations, reducing overhead but potentially delaying critical updates. Conversely, a smaller batch size allows for more frequent integration but increases complexity.

To illustrate, consider a scenario where you have migrations 1, 2, and 3. Migration 4 depends on migration 3. If migration 3 is merged into the main branch before migration 4 can be reviewed, it might introduce conflicts. With require-up-to-date enabled, migration 4 would only proceed if migration 3 has been fully integrated.

Now let's reason through how to tune the batch size:

Identify Critical Migrations: Determine which migrations are critical and should always be up-to-date before merging new changes.
Set Batch Size: For a small team, a smaller batch size ensures that each PR is thoroughly reviewed and conflicts are caught early. For larger teams or more complex projects, a larger batch size can reduce merge operations but might delay necessary updates.
Automated Checkpoints: Introduce automated checkpoints where the system pauses and checks if all required migrations have been applied before proceeding.

Here's an example to illustrate:

Suppose you have migrations M1, M2, and M3, with M3 dependent on both M1 and M2. If you set a batch size of 2, M3 would only be allowed to proceed after both M1 and M2 are merged.
Conversely, if the batch size is 3, all three migrations need to be fully integrated before any new changes can be merged.

In summary, a merge queue plus require-up-to-date ensures that no PR is merged against a stale base branch by waiting for all required migrations to be applied first. Tuning the batch size requires balancing between efficiency and thoroughness based on project requirements and team dynamics.

**Comparison:**

```
Migrations	Applied
1	Yes
2	Yes
3	Yes
4 (depends on 1 and 2)	Pending
5, 6	Not applied yet
```

**Code:**

```python
# Example of a simple check function in Python
def require_up_to_date(pr_branch, base_branch):
    # Simulate checking if all required migrations are present
    if not is_base_up_to_date(base_branch):
        return False
    else:
        return True

def is_base_up_to_date(branch):
    # Check if the branch has all necessary migrations applied
    return apply_migrations(branch)
```

**Example input / output:**

```

Given the input where migration 4 depends on migrations 1 and 2:

Input:

JSON
Copy
{
  "migrations": [
    {"id": 1, "applied": true},
    {"id": 2, "applied": true},
    {"id": 3, "applied": true}
  ]
}

Output (for batch size of 2):

JSON
Copy
{
  "merged_prs": ["4"],
  "remaining_migrations": []
}

Output (for batch size of 3):

JSON
Copy
{
  "merged_prs": [],
  "remaining_migrations": ["5", "6"]
}
```

**Data:**

```json
{
  "migrations": [
    {"id": 1, "applied": true},
    {"id": 2, "applied": true},
    {"id": 3, "applied": true}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Queue PR
4. Check Base Up-to-Date
5. Is Base Up-to-Date?
6. Wait for Required Migrations
7. Merge PR

