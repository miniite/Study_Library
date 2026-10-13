---
type: "practice"
difficulty: "Expert"
---

# A busy main branch takes 50 PRs a day and PRs keep breaking main because they were green against stale bases. Explain what a merge queue solves that required-up-to-date checks alone cannot, and how you'd tune batch size.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrency Groups and Merge Queues](<../../../../TOOLS/Git_and_GitHub/Git_Core/Concurrency_Groups_and_Merge_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

A merge queue addresses the issue of pull requests (PRs) being merged into a main branch before their bases are fully up-to-date by ensuring PRs are only merged once they have been rebased against the latest changes in the main branch. This prevents breakage caused by stale or outdated base branches.

#### Triaging the Question

The question revolves around understanding why required-up-to-date checks alone aren't sufficient to prevent broken builds, and how a merge queue can solve this problem. The key points are:

PRs keep breaking the main branch.
Required-up-to-date checks are insufficient.
We need to tune batch size for an effective merge queue.

#### First Principles Solution
Problem Understanding
Stale Bases: PRs often rely on a base branch that is out of date with the latest changes in main.
Breakage: When these stale PRs are merged, they cause failures because their codebase has diverged too much from main.
What Required-Up-To-Date Checks Can't Solve
Timing Issue: Required-up-to-date checks only ensure that a PR's base branch is up to date at the moment of the check. This doesn’t guarantee that by the time the PR is merged, the base has not further changed.
Merge Window: Changes in main can happen rapidly, making it impossible for PRs to always stay perfectly up-to-date.
What a Merge Queue Solves
Synchronization: A merge queue ensures that all PRs are rebased against main before merging them. This guarantees they are as current as possible.
Controlled Merges: By limiting the batch size, you can control how many PRs are processed in parallel, allowing for more thorough checks and better management of merge conflicts.
Tuning Batch Size

Batch Size Impact:

Small Batches (e.g., 5-10 PRs): More frequent merges mean less time between changes in main and the merging process, reducing staleness.
Large Batches (e.g., 20+ PRs): Fewer frequent merges can lead to higher risk of conflicts but might be more efficient in terms of Git operations.

Strategic Considerations:

Conflict Management: Larger batches increase the likelihood and complexity of merge conflicts, which need to be managed.
Team Workflow: Smaller batches may disrupt ongoing work if merges are too frequent or disruptive.
Resource Utilization: More frequent rebasing can strain Git servers and network resources.

#### Implementation
Strategy

Queue Management:

Implement a queue that holds PRs awaiting merge until they are rebased against the latest main.
Use automated triggers to rebase PRs in the queue periodically (e.g., every 2 hours).

Merge Strategy:

Merge up to a specified batch size at a time.
Optionally, allow manual merges for critical features or large changes.

Monitoring and Feedback:

Monitor merge outcomes to identify patterns of breakage or frequent conflicts.
Provide clear feedback to PR authors about why their changes are being held in the queue.

#### Closing Summary
+ Common Mistakes/Follow-Ups

A merge queue is a robust solution that ensures PRs are as up-to-date as possible before merging them into main, thus reducing the risk of breakage. Tuning batch size involves balancing between frequent, small merges (to minimize staleness) and less frequent, larger merges (to manage conflicts effectively). Common mistakes include:

Overly large batches leading to more merge conflicts.
Not rebasing frequently enough, which can still result in stale PRs.

For follow-ups, consider how to integrate CI/CD pipelines to further enhance the stability of merged code, or explore automated tools and scripts that can help manage the queue more efficiently.

**Comparison:**

```
Batch Size	Rebase Frequency (hours)	Potential Conflict Rate
5	2	Low
10	2	Moderate
20	4	High
```

**Code:**

```python
# Example Python script for managing a merge queue
import time

def rebased_prs(pr_queue):
    # Simulate rebase process
    print("Rebasing PRs against latest main...")
    time.sleep(5)  # Simulate delay
    return [f"rebased_{pr}" for pr in pr_queue]

def merge_prs(pr_queue, batch_size=10):
    rebased_prs_list = rebased_prs(pr_queue)
    merged_prs = []
    for i in range(0, len(rebased_prs_list), batch_size):
        print(f"Merging {i} to {i+batch_size}")
        time.sleep(3)  # Simulate merge
        merged_prs.extend(rebased_prs_list[i:i+batch_size])
    return merged_prs

pr_queue = ["PR1", "PR2", "PR3", "PR4", "PR5"]
merged_prs = merge_prs(pr_queue)
print("Merged PRs:", merged_prs)
```

**Example input / output:**

```text
Input:
- pr_queue = ["PR1", "PR2", "PR3", "PR4", "PR5"]
- batch_size = 10

Output:
- Merging 0 to 10
- Merged PRs: ['rebased_PR1', 'rebased_PR2', 'rebased_PR3', 'rebased_PR4', 'rebased_PR5']
```

**Data:**

```json
{
  "pr_queue": [
    "PR1",
    "PR2",
    "PR3",
    "PR4",
    "PR5"
  ],
  "batch_size": 10,
  "merged_prs": []
}
```

**Flow:**

1. PR Queue
2. Batch Size
3. 5 PRs
4. Merge & Rebase
5. Monitor Conflicts
6. 10 PRs
7. Merge & Rebase
8. Monitor Conflicts
9. 20+ PRs
10. Merge & Rebase
11. Monitor Conflicts

