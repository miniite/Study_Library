---
type: "practice"
difficulty: "Deep"
---

# You want to enforce that no migration file over a size threshold (a giant generated bulk-insert) or matching a connection-string pattern ever reaches the remote. Compare a server-side pre-receive hook, GitHub push protection, and a client-side hook, and the gap each leaves.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Server-side Hooks and Push Protection](<../../../../TOOLS/Git_and_GitHub/Git_Core/Server-side_Hooks_and_Push_Protection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce size thresholds and connection string patterns on migration files before they reach the remote, we can use three different approaches: server-side pre-receive hooks, GitHub push protection, and client-side hooks. Each method has its own strengths and weaknesses.

A server-side pre-receive hook is a script that runs on the Git server just before the push operation completes. It allows you to validate files or reject pushes based on certain criteria. On the other hand, GitHub push protection provides a web-based interface to define size restrictions directly in GitHub's UI. Finally, a client-side hook involves running a script locally that prevents pushing large files or files matching specific patterns.

In this answer, we will compare these methods and highlight their gaps in ensuring compliance.

**Comparison:**

```
Method	Description	Pros	Cons
Server-side pre-receive hook	Runs on Git server to validate files before push.	No network latency, flexible validation logic.	Requires access to Git server; complex error handling required.
GitHub Push Protection	Web-based UI in GitHub to define size restrictions.	Easy setup, automatic enforcement.	Limited by GitHub's implementation, no direct client-side control.
Client-side pre-push hook	Runs locally before push, can be customized easily.	Flexible and fast validation; fine-grained control.	May encounter network latency; limited access during development.
```

**Code:**

```
Example of server-side pre-receive hook (Git)

#!/bin/sh

if [ "$1" == "refs/heads/main" ]; then # Check if it's a push to main branch while read oldrev newrev refname; do sha=$(git rev-parse --short $refname^{tree}) # Get tree SHA of the reference for file in $(git diff-tree -r --no-commit-id --name-only $oldrev $sha | grep '.sql$'); do if [ "$(du -m "$file" | awk '{print $1}')" -gt "50" ]; then # Check file size > 50MB echo "File $file is too large (>50MB). Push denied." exit 1 fi done done fi

exit 0

CODE
Copy

```bash
# Example of client-side pre-push hook (Git)
#!/bin/sh

git rev-parse --abbrev-ref HEAD > /dev/null  # Check if a valid branch is being pushed to

for file in $(git diff-index --name-only HEAD); do
    if [[ $file =~ "connection_string_pattern" ]]; then  # Check for connection string pattern
        echo "File contains forbidden connection string pattern."
        exit 1
    fi
done

exit 0
YAML
Copy
# Example of GitHub Push Protection settings (via UI)
{
  "rules": [
    {
      "pattern": "^.*\\.(sql)$",  # Pattern to match SQL files
      "size_limit": "50MB"        # Size limit in MB
    }
  ]
}
```

