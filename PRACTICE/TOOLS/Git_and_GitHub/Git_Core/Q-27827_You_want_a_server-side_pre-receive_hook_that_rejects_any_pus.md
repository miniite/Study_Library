---
type: "practice"
difficulty: "Expert"
---

# You want a server-side pre-receive hook that rejects any push adding a migration file whose SQL matches a destructive pattern (DROP DATABASE, DELETE without WHERE) unless a bypass label is present. How do you implement it efficiently and what edge cases (force-push, deletes) break it?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Server-side Hooks and Push Protection](<../../../../TOOLS/Git_and_GitHub/Git_Core/Server-side_Hooks_and_Push_Protection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to create a server-side pre-receive hook in Git using git hooks that checks incoming pushes to detect if any migration files are being added or modified with destructive SQL patterns (such as DROP DATABASE, DELETE WITHOUT WHERE). If such files are found and no bypass label is present, the push should be rejected. We'll walk through the implementation details step by step.

This problem involves a few key steps:

Understanding the Problem: The pre-receive hook needs to inspect every file being pushed for destructive SQL patterns.
Setting Up the Environment: Ensure that the necessary tools and environment are set up correctly.
Implementing the Hook Logic: Write a script that parses incoming changes, checks for destructive patterns in migration files, and rejects the push if needed.
Handling Edge Cases: Consider scenarios such as force-pushes and deletions.

Let's start by setting up our environment:

Step 1: Setting Up the Environment

First, ensure you have Git installed on your server where the repository resides. You need to enable hooks in your repository. Typically, this involves placing a custom hook file in the hooks directory of your repository.

Step 2: Implementing the Hook Logic

We will write a pre-receive hook using a shell script (assuming Linux environment). Here's a high-level overview:

Read Incoming Changes: The hook receives as input the list of changes being pushed.
Parse Changes and Check Patterns: For each file, check if it is a migration file by its filename or extension. Then, use regular expressions to search for destructive SQL patterns.
Handle Bypass Label: If a bypass label exists in the commit message, allow the push.
Step 3: Writing the Hook Script

Here's an example of how you might write such a hook script:

BASH
#!/bin/bash

# Define destructive patterns
DESTRUCTIVE_PATTERNS=("DROP DATABASE" "DELETE.*WHERE")

# Function to check if file contains destructive pattern
contains_destructive_pattern() {
  local file="$1"
  for pattern in "${DESTRUCTIVE_PATTERNS[@]}"; do
    if grep -qE "$pattern" "$file"; then
      return 0 # Found a match
    fi
  done
  return 1 # No match found
}

# Process each incoming change
while read oldrev newrev refname; do
  echo "Processing push of $refname from $oldrev to $newrev"

  # Parse the files being added or modified in this push
  git diff-tree --no-commit-id --name-only -r $oldrev -- | while read file; do
    if [[ "$file" =~ \.sql$ ]]; then
      echo "Processing migration file: $file"

      if contains_destructive_pattern "$file"; then
        # Check for bypass label in commit message
        commit_message=$(git log -1 --format=%B)
        if ! grep -qE 'bypass' <<< "$commit_message"; then
          echo "Push rejected: Destructive SQL pattern found without bypass label."
          exit 1
        fi
      fi
    fi
  done

  # If no destructive patterns were found or bypassed, allow the push
  echo "Push allowed."
done

echo "All checks passed. Pushing changes..."
Step 4: Handling Edge Cases

#### Cons
ider edge cases:

Force-Pushes: Force-pushes might bypass our hook if they are pushed directly without going through a regular commit process.
Deletions: If files being deleted contain destructive patterns, we should consider removing them from the checks.

To handle force-pushes, ensure that your Git server enforces policies to prevent direct force-pushes. For deletions, update the script to ignore or validate these cases appropriately.

#### Closing Summary

This implementation efficiently checks for destructive SQL patterns in migration files and rejects pushes unless a bypass label is present. Edge cases like force-pushes and deletions need additional consideration to ensure robust behavior.

#### Common mistakes
include:

Failing to check all destructive patterns thoroughly.
Ignoring edge cases such as force-pushes or deletions.
Not properly handling the commit message to detect bypass labels.

