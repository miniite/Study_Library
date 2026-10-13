---
type: "practice"
difficulty: "Basic"
---

# How would you use a branch to add a new column with a default value while keeping the stable schema untouched on main?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Branching Strategies](<../../../../TOOLS/Git_and_GitHub/Git_Core/Branching_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how to use Git and GitHub branches effectively, let's focus on adding a new column with a default value in your database schema without altering the stable version on the main branch. This scenario involves using feature branches to make isolated changes.

#### Triaging the Question

The question is asking about managing database schema modifications using Git branches to avoid impacting the production or mainline codebase until the change is thoroughly tested and ready for release. The key concepts here are:

Branching: Creating a new branch from main to work on a specific feature.
Database Schema Changes: Modifying the database schema by adding columns with default values.
Staying Stable on Main: Ensuring that main remains stable and unaffected until the feature is fully tested.

#### Assumptions
You are familiar with basic Git operations like creating branches, committing changes, pushing to GitHub, and merging back.
The database schema modification requires running SQL scripts or migrations.

#### First Principles Solution

Create a New Feature Branch:

Start by checking out the main branch to ensure you're working on the latest stable version.
Create a new feature branch, e.g., add-new-column.

Modify the Database Schema:

Use SQL scripts or migration tools (like Alembic for Python applications) to add a new column with a default value in your database.

Commit and Push Changes:

Commit your changes locally.
Push the branch to GitHub to collaborate with other team members if needed.

Test Thoroughly:

Run tests to ensure that adding the new column does not break any existing functionality.

Merge Back into Main:

Once tested, create a pull request (PR) from add-new-column to main.
Review and merge the PR after approval.
Ensure main is merged back successfully without issues.

#### Closing Summary

By using Git branches effectively, you can make isolated changes like adding columns with default values in your database schema. This approach keeps the stable version on the main branch untouched until the feature is fully tested and ready for release.

#### Common Mistakes/Follow-Ups
Failing to Create a Separate Branch: Directly making changes on main can introduce instability.
Forgetting to Test Thoroughly: Rushing through testing might lead to bugs that only appear in production.
Not Using Pull Requests Properly: Merging directly into main without review can cause merge conflicts or overlooked issues.

**Code:**

```sql
-- Example SQL script for adding a new column with a default value
ALTER TABLE users ADD COLUMN age INT DEFAULT 0;
```

