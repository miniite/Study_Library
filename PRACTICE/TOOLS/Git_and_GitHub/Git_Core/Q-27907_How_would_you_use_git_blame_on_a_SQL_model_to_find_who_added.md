---
type: "practice"
difficulty: "Intermediate"
---

# How would you use git blame on a SQL model to find who added a WHERE filter that is now excluding rows the report needs?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Blame and Ignore Revisions](<../../../../TOOLS/Git_and_GitHub/Git_Core/Blame_and_Ignore_Revisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, we need to understand how git blame works and its application in identifying the author of code changes, especially in SQL models. The key idea here is that by using git blame, we can pinpoint exactly when a change was made and who made it, which will help us identify the person responsible for adding a WHERE filter that excludes important rows from our reports.

#### Triage

The question is essentially asking how to use Git’s blame command to trace back a specific code change (adding a WHERE clause) in an SQL model. The interviewer might be probing your understanding of version control systems, SQL queries, and the ability to correlate changes with their impact on downstream processes like reports.

#### First-Principles Solution
Identify the Change: Locate the SQL query where the WHERE filter has been added.
Use git blame: Run the command to see who made each line of code leading up to the introduction of the new WHERE clause.
Analyze the History: Examine the commit history around that change, noting any additional context about why this change was introduced.
Step-by-Step Solution
Locate the Query: First, find the specific SQL query or model where the issue is occurring. This could be part of a larger data pipeline or database schema.
Run git blame:
BASH
git blame -L <start-line>,<end-line> path/to/sql/model.sql
Replace <start-line> and <end-line> with the lines where you suspect the change occurred, or just use the whole file if unsure.
Interpret the Output: The git blame command will output each line of the SQL script along with its commit hash and author information:
BASH
123456789abcdef0123456789abcdef0123456789 Author Date Line Content

#### Closing Summary

By using git blame, we can trace the history of our SQL model and pinpoint exactly who added the WHERE filter that is now causing issues in our reports. This process helps us understand not just when, but also why the change was made, which is crucial for maintaining code quality and collaboration.

#### Common Mistakes/Follow-Ups
Misinterpreting git blame Output: Ensure you correctly identify the relevant lines.
Ignoring Contextual Information: Remember to review commit messages and other context around the change.
Not Exploring Branch History: Sometimes, changes might have been introduced in a branch that was later merged. Checking merge commits can provide additional insight.

**Code:**

```bash
git blame -L 100,200 path/to/sql/model.sql
```

