---
type: "practice"
difficulty: "Deep"
---

# Your team enforces server-side push protection, but a determined dev could still commit a secret locally. Compare a pre-receive hook, GitHub push protection, and client-side hooks for blocking an AWS key, including the gap each leaves.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Server-side Hooks and Push Protection](<../../../../TOOLS/Git_and_GitHub/Git_Core/Server-side_Hooks_and_Push_Protection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of preventing local commits containing secrets (specifically, AWS keys) from being pushed to a remote repository, we need to compare three main approaches: pre-receive hooks, GitHub push protection, and client-side hooks. Each method has its strengths and weaknesses, which I will outline in this answer.

A pre-receive hook runs on the server before the commit is accepted by Git; a GitHub push protection policy enforces rules at the repository level via the GitHub UI; and client-side hooks run locally before any push command is executed. Let's examine each approach in detail.

**Comparison:**

```
	Pre-receive Hook	GitHub Push Protection	Client-side Hook
Location	Server-side	Repository settings	Client-side
Execution Time	Before commit is accepted	When push request is made	Before any push command
Enforceability	Definite (if configured)	Optional, depends on admin	Can be bypassed
Visibility	Server logs	GitHub UI	Console output
Flexibility	Highly flexible	Limited by repository settings	Can be misconfigured
Complexity	Medium	Low to moderate	High
```

**Code:**

```
Example of a pre-receive hook script

#!/bin/sh

for ref in $.git_push_refs; do # Check if the push contains the AWS key pattern if git log --format=%B $old_commit_id..$new_commit_id | grep -q 'AWS_ACCESS_KEY_ID' ||
git diff --name-only $old_commit_id $new_commit_id | grep -E '^secrets/.*/aws_key.txt$' > /dev/null; then echo "Push blocked: AWS key detected in commit." exit 1 fi done

CODE
Copy
```bash
# Example of a client-side hook script (pre-push)
#!/bin/sh

if git rev-parse --verify HEAD > /dev/null 2>&1; then
    against=HEAD
else
    # initial commit
    against=origin/$(git symbolic-ref --short -q HEAD)
fi

# Check if the push contains the AWS key pattern
if git diff --name-only $against | grep -E '^secrets/.*/aws_key\.txt$' > /dev/null; then
    echo "Prevent local commits containing AWS keys from being pushed."
    exit 1
fi
```

**Example input / output:**

```text
# Example pre-receive hook input -> output
Input: Pre-receive hook with AWS key pattern check
Output: Push blocked if the commit contains the AWS key

# Example client-side hook input -> output
Input: Client-side pre-push hook with AWS key pattern check
Output: Local commit containing AWS key prevents push
```

**Data:**

```json
{
  "pre_receive_payload": {
    "commit_id": "abc123",
    "ref_name": "refs/heads/main"
  },
  "client_push_payload": {
    "old_commit_id": "def456",
    "new_commit_id": "ghi789"
  }
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD
3. A[User commits locally] --> B[Client-side hook runs]
4. B --> C[Commits checked for AWS keys]
5. C --> D[Valid commit, push allowed] OR E[Invalid commit, local commit blocked]
6. E --> F[User tries to push]
7. F --> G[Pre-receive hook runs on server]
8. G --> H[Push contents checked against AWS key patterns]
9. H --> I[Valid push, accepted by server] OR J[Invalid push, rejected by server]

