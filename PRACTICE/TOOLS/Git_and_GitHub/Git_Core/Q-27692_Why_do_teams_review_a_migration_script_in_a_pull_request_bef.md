---
type: "practice"
difficulty: "Basic"
---

# Why do teams review a migration script in a pull request before it is ever run against a real database?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Pull Requests](<../../../../TOOLS/Git_and_GitHub/Git_Core/Pull_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Teams review migration scripts in a pull request (PR) to ensure they are correct, safe, and will not cause any data loss or corruption. This practice involves several key steps from first principles:

Purpose of Review: The main goal is to catch bugs early before the script is executed against a production database.
Safety and Data Integrity: Ensuring that the migration script does not accidentally delete, corrupt, or modify critical data.
Collaborative Learning: Peer review helps developers learn from each other's practices and code.
Version Control and Traceability: Keeping track of changes and maintaining a history of what was done and why.

**Code:**

```
MARKDOWN
Copy
- A typical PR might include:
  - A brief description of the changes in the commit message.
  - The migration script itself, often formatted for readability.
  - Any supporting documentation or notes on how to test the migration.
```

