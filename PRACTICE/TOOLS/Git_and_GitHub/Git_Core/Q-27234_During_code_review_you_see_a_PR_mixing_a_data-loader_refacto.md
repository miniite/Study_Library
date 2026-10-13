---
type: "practice"
difficulty: "Intermediate"
---

# During code review you see a PR mixing a data-loader refactor, a new feature, and a metric-logging fix in one commit. Explain concretely why this hurts revertability and bisect for a production model, and how you'd coach the author to restructure it.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Atomic Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Atomic_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

When a pull request (PR) mixes multiple unrelated changes in one commit, such as a data-loader refactor, a new feature, and a metric-logging fix, it significantly reduces the PR's revertability and bisectability. This makes debugging and maintaining the production model more challenging. I would coach the author to split these changes into separate commits for clarity, ease of review, and better management.

#### Triaging the Question

The core issue here is about code quality and maintainability. Specifically, the question is probing your understanding of version control best practices, particularly in Git and GitHub. The assumptions are that the production model relies on reliable change management techniques to ensure robustness and traceability.

#### First-Principles Solution

Revertability:

When a PR contains multiple changes, it becomes difficult to understand which part of the code caused a regression or issue if something goes wrong in production.
For example, let's say the data-loader refactor introduced a bug that broke model predictions. If this change is mixed with a new feature and metric-logging fix, it would be hard to isolate whether the bug was due to the refactoring or another component.

Bisectability:

Bisecting refers to finding the exact commit where a regression started in production.
If changes are tightly coupled together, bisecting becomes more complex and time-consuming. For instance, if you need to revert only the data-loader refactor but it's mixed with other features, reverting might require additional work.

Code Review:

Code reviews become less effective when multiple unrelated changes are bundled in one PR.
It’s harder for reviewers to focus on each change independently, which can lead to missed bugs or oversights during the review process.

Separation of Concerns:

Each commit should ideally have a single responsibility (SRP). This makes it easier to understand what each change is doing and how it fits into the larger project.
By separating changes, you ensure that when issues arise, they can be isolated more easily, making debugging faster.
Coaching Strategy:

Explain the Importance:

Start by explaining why code separation is important for maintainability and ease of debugging.
Use an analogy: "Imagine if a chef mixed all their ingredients into one bowl before baking – it would be hard to pinpoint which ingredient caused a problem if something went wrong."

Demonstrate Best Practices:

Show how they can split the PR into separate commits using Git commands like git commit and git rebase.
Provide an example of how to structure the code in a way that clearly separates concerns.

Highlight Benefits:

Reiterate the benefits of clear, isolated changes: easier review, better revertability, and more effective debugging.
Emphasize that this practice not only helps with current PRs but also sets a standard for future contributions to the project.

Encourage Testing:

Ensure they understand the importance of writing tests for each change individually before merging them back into the main branch.
Stress that unit tests and integration tests are crucial in verifying the correctness of each component.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
Summary: Mixing unrelated changes in one commit can lead to decreased revertability and bisectability, making it harder to manage and debug production issues. It's important to maintain clear separation of concerns through well-defined commits.

#### Common Mistakes
Ignoring code structure for the sake of speed.
Assuming that mixed changes are easier to review or manage.

#### Follow-Ups
What tools or practices do you use in your workflow to ensure good commit history?
How do you handle complex refactoring tasks while still maintaining separation of concerns?

**Comparison:**

```
Commit	Message	Files
1	Refactor data-loader to improve performance	dataloader.py
2	Add new feature for improved model accuracy	new_feature_code.py
3	Fix metric-logging issue for better monitoring	logging_metrics.py
```

**Code:**

```bash
# Example of splitting a PR into multiple commits

# Initial setup:
git add .
git commit -m "Initial commit with refactor, new feature and metric logging"

# Refactor the data-loader
git checkout master  # Ensure you are on the correct branch
git add dataloader.py
git commit -m "Refactor data-loader to improve performance"

# Add new feature
git add new_feature_code.py
git commit -m "Add new feature for improved model accuracy"

# Fix metric-logging issue
git add logging_metrics.py
git commit -m "Fix metric-logging issue for better monitoring"
```

**Example input / output:**

```text
# Example input-output of bisecting a problematic commit

Initial state:
git bisect start
git bisect bad # Assume the current version is known to be broken

Bisecting...
(bisection process continues until the exact commit causing the issue is found)

Final result:
git bisect good 1234567890abcdef # Mark the first non-broken commit
```

**Data:**

```json
{
  "commit1": {
    "message": "Refactor data-loader to improve performance",
    "files": ["dataloader.py"]
  },
  "commit2": {
    "message": "Add new feature for improved model accuracy",
    "files": ["new_feature_code.py"]
  },
  "commit3": {
    "message": "Fix metric-logging issue for better monitoring",
    "files": ["logging_metrics.py"]
  }
}
```

**Flow:**

1. No
2. Yes
3. Initial Code
4. Are Changes Separated?
5. Merge All Changes
6. Bisect to Find Issue
7. Review and Merge Separately
8. Monitor for Issues

