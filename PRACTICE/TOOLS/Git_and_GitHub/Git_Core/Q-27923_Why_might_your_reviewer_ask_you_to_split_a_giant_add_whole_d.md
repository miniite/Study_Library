---
type: "practice"
difficulty: "Basic"
---

# Why might your reviewer ask you to split a giant 'add whole data mart' commit into smaller commits per model or layer?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Commit Messages](<../../../../TOOLS/Git_and_GitHub/Git_Core/Commit_Messages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git and GitHub, reviewers often prefer that changes be broken down into smaller, more manageable chunks. This is particularly true when a developer adds an entire data mart at once. The rationale behind splitting such commits into smaller ones per model or layer lies in several key benefits. First, it makes the commit history cleaner and easier to understand. Second, it simplifies the process of rolling back changes if something goes wrong. Lastly, it allows for more focused reviews, ensuring that each component is thoroughly checked before being merged.

In this context, a reviewer might ask you to split your 'add whole data mart' commit into smaller commits per model or layer because:

Reduced Complexity: Smaller commits are easier to understand and review.
Easier Debugging: If an issue arises after the merge, it's much simpler to pinpoint which part of the code introduced the bug.
Incremental Validation: Each commit can be validated independently, ensuring that each component works as expected before moving on to the next one.

To illustrate this concept, let’s consider a simple example where you are adding a data mart for an e-commerce application with multiple models (e.g., users, products, and orders).

#### Triaging the Question

The reviewer's request is essentially asking you to follow best practices in Git commit management. The key assumptions here include:

You understand that maintaining clean, modular code is essential.
You recognize that a single large commit can obscure changes and make debugging difficult.

#### First-Principles Solution
Identify Models/Layers: Break down the data mart into its constituent models or layers (e.g., users, products, orders).
Commit Separately: Commit each model or layer separately, ensuring that each commit adds value and is logically coherent.
Review Each Component: Allow your reviewer to review each component independently, providing a clear and detailed history of changes.

#### Example
Workflow

Here’s an example of how you might approach this:

MARKDOWN
1. `git checkout -b add_users`
2. Add the users model and commit: `git commit -m "Add Users Model"`
3. Reviewer checks: `git diff HEAD~1 HEAD`
4. Merge if approved: `git push origin add_users`
5. Repeat for other models:
   - `add_products` (commit, review, merge)
   - `add_orders` (commit, review, merge)

#### Closing Summary
and Common Mistakes

By splitting the commit into smaller chunks per model or layer, you ensure that each change is clearly documented and can be reviewed independently. This approach not only adheres to best practices but also makes it easier to manage complex changes.

#### Common mistakes
include:

Failing to break down large commits: This can lead to a convoluted history.
Not allowing sufficient time for review: Hasty merges without thorough checks can introduce bugs.

**Code:**

```
SH
Copy
# Example Git Workflow
git checkout -b add_users
touch users/*.py
git commit -m "Add Users Model"
# Review and merge
git push origin add_users

git checkout -b add_products
touch products/*.py
git commit -m "Add Products Model"
# Review and merge
git push origin add_products

git checkout -b add_orders
touch orders/*.py
git commit -m "Add Orders Model"
# Review and merge
git push origin add_orders
```

**Flow:**

1. Commit and Review
2. Commit and Review
3. Commit and Review
4. Initial State
5. Add Users Model
6. Merge to Main
7. Add Products Model
8. Merge to Main
9. Add Orders Model
10. Merge to Main

