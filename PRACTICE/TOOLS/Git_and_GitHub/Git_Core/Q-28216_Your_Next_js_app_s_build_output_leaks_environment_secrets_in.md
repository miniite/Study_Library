---
type: "practice"
difficulty: "Intermediate"
---

# Your Next.js app's build output leaks environment secrets into the bundle occasionally. How would you use gitleaks or push-protection plus pre-commit hooks to stop secrets reaching GitHub, and what's the gap client-side hooks leave?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Server-side Hooks and Push Protection](<../../../../TOOLS/Git_and_GitHub/Git_Core/Server-side_Hooks_and_Push_Protection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of environmental secrets leaking into your Next.js app’s build output, we can leverage tools like gitleaks for static analysis and push protection to prevent pushing secret-containing commits to GitHub. Additionally, we can use pre-commit hooks to enforce security policies before any code is committed. However, it's important to note that client-side hooks alone might not fully cover the issue, as they do not inspect the build output directly.

**Code:**

```
SH
Copy
# Example pre-commit hook to check for secrets using gitleaks

#!/bin/bash
# Install gitleaks if not installed
command -v gitleaks &> /dev/null || { echo "gitleaks is required but it's not installed. Aborting." >&2; exit 1; }

# Check the current branch and skip non-protected branches (e.g., master, main)
current_branch=$(git symbolic-ref --short HEAD)
if [[ $current_branch == "master" ]] || [[ $current_branch == "main" ]]; then
    echo "Skipping pre-commit hook on protected branch: $current_branch"
    exit 0
fi

# Run gitleaks to check for secrets in the staged files
gitleaks -v --json | jq '.[].matches' > /tmp/gitleaks_output.json

if [ -s "/tmp/gitleaks_output.json" ]; then
    echo "Secrets detected in staged files:"
    cat /tmp/gitleaks_output.json
    exit 1
else
    echo "No secrets found. Commit allowed."
fi
```

**Example input / output:**

```text
# Example of running the pre-commit hook before a commit
$ git add . && gitleaks-check.sh
Secrets detected in staged files:
[
  {
    "line": 123,
    "column": 45,
    "context": "export const API_KEY = 'secret'",
    "secrets": [
      {
        "type": "API",
        "value": "secret"
      }
    ]
  }
]

# Git will refuse to commit if secrets are detected
```

**Flow:**

1. Yes
2. No
3. Secrets found
4. No secrets found
5. Start
6. Is the current branchprotected?
7. Skip pre-commit hook
8. Run gitleaks to check forsecrets
9. Abort commit and displayerrors
10. Commit allowed

