---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a simple bash pre-commit hook that blocks a commit if it contains the word 'password' or an AWS key pattern?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Server-side Hooks and Push Protection](<../../../../TOOLS/Git_and_GitHub/Git_Core/Server-side_Hooks_and_Push_Protection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this requirement, we need to create a pre-commit hook in Git that will prevent commits from being pushed if they contain specific sensitive information such as "password" or an AWS key pattern. This involves writing a bash script that checks the commit content before allowing it to be committed.

The core idea is to monitor the files being staged for commit, check their contents using grep, and abort the commit process if any of these patterns are found.

Here's how we can approach this step-by-step:

Understand the Problem: We need a hook that triggers before each Git commit.
Identify Patterns to Block: The script should look for "password" or AWS key patterns in files being staged.
Write the Bash Script: Create and place the script in the appropriate location within your Git repository.

Let's dive into the solution:

#### Triage

The question is asking us to implement a pre-commit hook that will enforce certain rules before allowing commits. Specifically, it needs to check for sensitive information like "password" or AWS keys in the staged files and block the commit if these patterns are found. We need to ensure this script works seamlessly within the Git workflow without interfering with normal operations.

#### First-Principles Solution

Location of the Hook:

Place the hook script at \.git\hooks\pre-commit. This ensures that the script runs before any commit.

Content of the Script:

BASH
#!/bin/bash

# Define patterns to match against
PASSWORD_PATTERN="password"
AWS_KEY_PATTERN="[a-zA-Z0-9]{16,}"  # Example pattern for 16 characters long string

# Loop through all files in the staging area
for file in $(git diff --cached --name-only --diff-filter=ACM)
do
    if [[ $(grep -E "$PASSWORD_PATTERN|$AWS_KEY_PATTERN" $file) ]]; then
        echo "Commit blocked: Sensitive information found in $file"
        exit 1
    fi
done

# If no sensitive information is found, allow the commit
exit 0

Explanation:

The script uses git diff --cached --name-only --diff-filter=ACM to list all files that have been added or modified in the current commit.
It then checks each file for the presence of either "password" or the AWS key pattern using grep.
If any match is found, it prints an error message and exits with a non-zero status, thus blocking the commit.

#### Closing Summary

This script effectively ensures that no commits containing sensitive information like passwords or AWS keys can be pushed to the repository. It operates by monitoring the staged changes in each commit, checking for specific patterns, and preventing the commit if any are found.

#### Common Mistakes/Follow-Ups
Misplaced Hook: Ensure the hook is correctly placed in \.git\hooks\pre-commit.
Pattern Matching: The pattern matching can be improved depending on the exact AWS key format you want to block.
Performance Consideration: Be mindful of performance for large repositories with many staged files. Optimizing the script or using a more efficient tool might be necessary.

**Code:**

```bash
#!/bin/bash

# Define patterns to match against
PASSWORD_PATTERN="password"
AWS_KEY_PATTERN="[a-zA-Z0-9]{16,}"  # Example pattern for 16 characters long string

# Loop through all files in the staging area
for file in $(git diff --cached --name-only --diff-filter=ACM)
do
    if [[ $(grep -E "$PASSWORD_PATTERN|$AWS_KEY_PATTERN" $file) ]]; then
        echo "Commit blocked: Sensitive information found in $file"
        exit 1
    fi
done

# If no sensitive information is found, allow the commit
exit 0
```

**Example input / output:**

```text
$ git add some_file.txt
$ echo "This is a test with password123" >> some_file.txt
$ git commit -m "Test commit"
Commit blocked: Sensitive information found in some_file.txt
```

