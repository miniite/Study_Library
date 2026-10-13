---
type: "practice"
difficulty: "Basic"
---

# Why should a compiled database backup file (.bak or .dump) usually be kept out of Git, and how would you exclude it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Gitignore](<../../../../TOOLS/Git_and_GitHub/Git_Core/Gitignore.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that your version-controlled project remains clean and manageable, it's crucial to exclude large binary files like compiled database backups from your Git repository. This practice not only keeps your Git history lean but also prevents accidental pushes of sensitive data.

TL;DR:

A compiled database backup file should be excluded from Git because it is too large for version control and could contain sensitive information. You can achieve this by adding the file to .gitignore.

#### Triage
What's Being Probed

The question aims to understand your knowledge of best practices in version control, particularly when dealing with large files that do not belong in a repository.

#### First-Principles Solution
Why Exclude Large Files?
Size: Compiled database backups are typically very large and unnecessary for the project history.
Sensitivity: These files often contain sensitive data such as credentials or personal information, which should be protected.
Steps to Exclude:
Add the file path to your .gitignore file.
Detailed Explanation:

Size Considerations:

Large backup files can significantly increase the size of your repository and slow down operations like cloning, pulling, and pushing.
For example, a 1GB backup file could quickly grow your repo by several gigabytes.

Security Concerns:

If sensitive data is included in version control, it risks exposure through accidental pushes or public repos. This can lead to serious security breaches.

#### Example
A .bak file might contain database credentials for a production environment, which should not be publicly accessible.

Gitignore Configuration:

Create or modify your .gitignore file in the root of your project directory if it doesn't already exist. Add the path to the backup file.
PLAINTEXT
# Example .gitignore entry
db-backup.bak
Alternatively, if you're using a common backup location:
PLAINTEXT
# Example .gitignore entry for common locations
/backups/*

#### Closing Summary
and Common Mistakes/Follow-Ups:
Summary: Excluding large binary files like database backups from Git is essential for maintaining repository size and security. Use .gitignore to define which files should be excluded.

#### Common Mistakes
Forgetting to add the file path to .gitignore, leading to accidental commits.
Pushing sensitive data, including credentials or personal information.

#### Follow-Up
Questions:
How do you handle versioning of database changes in Git?
What are the best practices for excluding files that change frequently but shouldn't be versioned?

**Code:**

```plaintext
# Example .gitignore entry
db-backup.bak
```

