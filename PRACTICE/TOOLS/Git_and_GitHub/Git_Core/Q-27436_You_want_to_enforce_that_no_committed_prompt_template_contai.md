---
type: "practice"
difficulty: "Deep"
---

# You want to enforce that no committed prompt template contains a hardcoded PII example or a real customer transcript. Compare a server-side pre-receive hook, GitHub push protection, and a client hook for this policy.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Server-side Hooks and Push Protection](<../../../../TOOLS/Git_and_GitHub/Git_Core/Server-side_Hooks_and_Push_Protection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of enforcing that no committed prompt template contains a hardcoded PII example or a real customer transcript, we need to compare three approaches: a server-side pre-receive hook, GitHub push protection, and a client-side hook. Each method has its own strengths and weaknesses in terms of enforcement, performance, and user experience.

#### Triaging the Question

The question is essentially asking us to evaluate different strategies for ensuring that code commits do not contain sensitive information such as Personally Identifiable Information (PII) or real customer transcripts. The interviewer is likely probing our understanding of how Git and GitHub handle commit policies, the trade-offs between server-side and client-side enforcement mechanisms, and the nuances of implementing security policies.

#### Assumptions
We are working with a team that uses Git/GitHub for version control.
PII examples or real customer transcripts should be avoided in committed code to protect sensitive data.
The solution must prevent such content from being pushed to the repository without manual intervention or automated checks.
Server-Side Pre-Receive Hook

A pre-receive hook is a script that runs on the remote Git server and can inspect changes before they are accepted into the repository. This approach ensures that only commits meeting the policy criteria are allowed.

#### Implementation
Hook Script: Create a shell or Python script to check each commit.
Script Logic: The script will read the contents of new commits, using regular expressions or custom parsing logic to detect PII examples or customer transcripts.
Policy Enforcement: If any forbidden content is detected, the hook should reject the push.

#### Example
Script
BASH
#!/bin/bash

# Define a list of patterns to match against
PII_PATTERNS=("customerID" "SSN" "creditCard")

for ref in $1; do
  for sha in $(git rev-list $ref); do
    if git show --no-patch --oneline $sha | grep -E "$(IFS='|'; echo "${PII_PATTERNS[*]}")"; then
      echo "Commit contains forbidden content. Please revise and resubmit."
      exit 1
    fi
  done
done

exit 0
GitHub Push Protection

GitHub's push protection feature allows setting up policies to enforce certain conditions before a push is allowed. This can include branch protection rules, which are more user-friendly than server-side hooks but still effective.

#### Implementation
Repository Settings: Navigate to the repository settings and enable branch protection.
Custom Checks: Define custom checks that run on each commit, such as running scripts or checking for specific patterns in commit messages.
Client-Side Hook (Pre-Commit)

A client-side pre-commit hook runs locally before a push is made. This ensures developers are aware of the policy and can fix issues before committing changes to the remote repository.

#### Implementation
Hook Script: Create a pre-commit script that checks for forbidden content.
Script Logic: Similar to the server-side hook, but runs on the developer's local machine.

#### Example
Script
BASH
#!/bin/bash

# Define a list of patterns to match against
PII_PATTERNS=("customerID" "SSN" "creditCard")

for file in $(git diff --cached --name-only --diff-filter=A); do
  if grep -E "$(IFS='|'; echo "${PII_PATTERNS[*]}")" $file; then
    echo "Found forbidden content in $file. Please revise."
    exit 1
  fi
done

exit 0
Comparison and Trade-offs
Pre-Receive Hook: Ensures security but requires server-side resources and administration.
GitHub Push Protection: User-friendly but may not cover all edge cases and can be bypassed if not configured properly.
Client-Side Hook: Local enforcement ensures immediate feedback to developers, but might require additional setup.

#### Closing Summary

In conclusion, each method has its own advantages. A combination of client-side pre-commit hooks and server-side pre-receive hooks would provide a robust solution that combines user feedback with security checks. GitHub push protection can be used as an additional layer to simplify enforcement without overloading the server.

#### Common Mistakes
and Follow-Ups
Overlooking Local Developers: Ensure all developers are aware of the policy and have the necessary tools.
Bypassing Checks: Make sure automated checks cannot be bypassed by force pushing.
Performance Overhead: Server-side hooks can introduce latency, so consider their impact on performance.

**Code:**

```bash
# Pre-receive hook example
#!/bin/bash

PII_PATTERNS=("customerID" "SSN" "creditCard")

for ref in $1; do
  for sha in $(git rev-list $ref); do
    if git show --no-patch --oneline $sha | grep -E "$(IFS='|'; echo "${PII_PATTERNS[*]}")"; then
      echo "Commit contains forbidden content. Please revise and resubmit."
      exit 1
    fi
  done
done

exit 0

# Pre-commit hook example
#!/bin/bash

PII_PATTERNS=("customerID" "SSN" "creditCard")

for file in $(git diff --cached --name-only --diff-filter=A); do
  if grep -E "$(IFS='|'; echo "${PII_PATTERNS[*]}")" $file; then
    echo "Found forbidden content in $file. Please revise."
    exit 1
  fi
done

exit 0
```

