---
type: "practice"
difficulty: "Basic"
---

# You changed a column's data type in a CREATE TABLE script. How would you record that as a clear, self-contained Git commit?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Versioning SQL and Migrations](<../../../../TOOLS/Git_and_GitHub/Git_Core/Versioning_SQL_and_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When making changes to database scripts like altering a column’s data type, it is crucial to document these changes properly using Git commits. For a fresher or an early-stage Database Developer (DB Dev) in this role, the key is to ensure that each commit is clear and self-contained. Here’s how you can record such a change as a clear Git commit:

#### Overview
A Git commit should be brief but descriptive enough for others to understand what was changed without needing to look at the full script. It should follow best practices like using meaningful commit messages.

Understanding the Question: The core of this question is about how you would document a specific change (altering a column's data type) in your database schema as part of your version control process. This involves identifying what exactly was changed, and how to properly communicate that change through a Git commit message.

#### First Principles Solution

Identify the exact change: In this case, we are changing a column’s data type.
Use clear language in the commit message: For example, "Change users.email from VARCHAR(255) to TEXT."
Ensure no unintended changes: Only include the necessary changes and avoid committing unrelated code.

Here’s an example of how you would write this Git commit:

BASH
git commit -m "Change users.email column data type from VARCHAR(255) to TEXT"

#### Closing Summary
Always aim for a concise yet descriptive message. This ensures that others can quickly understand the change just by looking at the commit history.

#### Common Mistakes/Follow-ups

Including unrelated changes in the same commit.
Not providing enough detail in the commit message, leading to confusion later.
Forgetting to stage and commit changes before pushing.

**Code:**

```bash
git commit -m "Change users.email column data type from VARCHAR(255) to TEXT"
```

