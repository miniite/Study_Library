---
type: "practice"
difficulty: "Basic"
---

# How would you view the commit history for a single stored-procedure file to see every change it has ever had?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Searching History](<../../../../TOOLS/Git_and_GitHub/Git_Core/Searching_History.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To view the commit history for a single stored procedure file in Git and GitHub, you'll want to understand how commits are recorded and tracked within version control systems like Git. This involves using Git commands to fetch historical information about specific files.

When dealing with database development, it’s crucial to track changes over time, especially since stored procedures can undergo frequent modifications during development and maintenance phases. Let's walk through the process step-by-step from first principles:

Identify the Stored Procedure File: Determine the exact path of the file within your project directory structure.
Use Git Commands to Fetch Commit History: Utilize Git commands to retrieve the commit history for that specific file.

Here’s how you would do it in practice:

#### Triaging the Question

The question is essentially asking about a method to view all changes made to a stored procedure over its entire lifecycle from within a Git repository. The key areas we need to cover include:

Understanding the command git log and its variants.
Using filters to focus on specific files.
Exploring additional options in Git like --follow for handling renames.
Step-by-Step Solution

Identify the File Path: Suppose your stored procedure is located at /src/stored_procedures/my_stored_procedure.sql.

Fetch Commit History with git log:

Open a terminal and navigate to your repository directory.
Use the command:
SH
git log --pretty=format:'%h %ad | %s%d [%an]' --date=short -- src/stored_procedures/my_stored_procedure.sql
This command will show you a list of commits affecting my_stored_procedure.sql with details like commit hash, date, summary, and author.

Filtering by File Renames:

Sometimes stored procedures might be moved or renamed over time.
Use the --follow option to track renames:
SH
git log --pretty=format:'%h %ad | %s%d [%an]' --date=short --follow src/stored_procedures/my_stored_procedure.sql

Interactive History: If you want a more interactive view, consider using gitk (GUI) or tig:

For GUI: Run gitk --srcdir=[path to your repository]/src/stored_procedures/my_stored_procedure.sql.
For CLI: Install and use tig with the command:
SH
tig src/stored_procedures/my_stored_procedure.sql

#### Closing Summary

In summary, viewing the commit history for a stored procedure involves using Git commands like git log to fetch detailed histories. Filters such as file paths and rename tracking options are essential tools in understanding how a specific piece of code has evolved over time.

#### Common Mistakes/Follow-Ups
Mistake: Not understanding that git log alone might not show renames, leading to incomplete history.

#### Follow-Up
Exploring other Git tools like tig for more interactive exploration and using git blame for line-level context during code reviews.

**Code:**

```
SH
Copy
# Example commands
git log --pretty=format:'%h %ad | %s%d [%an]' --date=short -- src/stored_procedures/my_stored_procedure.sql
git log --pretty=format:'%h %ad | %s%d [%an]' --date=short --follow src/stored_procedures/my_stored_procedure.sql
```

