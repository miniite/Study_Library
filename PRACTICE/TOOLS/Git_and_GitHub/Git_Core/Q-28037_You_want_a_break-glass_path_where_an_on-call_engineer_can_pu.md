---
type: "practice"
difficulty: "Expert"
---

# You want a break-glass path where an on-call engineer can push an emergency pipeline fix to prod but every override leaves an immutable, reviewed-after-the-fact audit trail in Git. Design the branch protection and reconciliation workflow, and how you'd audit those overrides monthly.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Branch Protection Design](<../../../../TOOLS/Git_and_GitHub/Git_Core/Branch_Protection_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the need for a break-glass path where an on-call engineer can push emergency pipeline fixes to production while maintaining an immutable and reviewed-after-the-fact audit trail, we'll design branch protection rules that enforce strict controls and then create a reconciliation workflow to audit these overrides. The key idea is to balance between enabling rapid response during emergencies and ensuring thorough post-incident reviews.

The core of the solution involves:

Defining Branch Protection Rules for master branches to ensure they are always protected.
Creating an Override Process that logs every emergency change, allowing for detailed audit after the fact.
Monthly Reconciliation Workflows to review and document all overrides.

**Comparison:**

```
Commit Message	Author	Date
Emergency fix for production issue #123	oncall-engineer	2023-09-15
Override Log Content	Details
a1b2c3d4e5: Emergency fix for prod #123 by oncall-engineer@example.com	2023-09-15T14:00Z
```

**Code:**

```python
# Example of a simple Git hook script to log changes
import os

def on_push():
    if os.environ.get('GIT_COMMIT_MSG').startswith("Emergency fix for"):
        with open('/path/to/override_log.txt', 'a') as log:
            log.write(f"{os.environ['GIT_COMMIT']}: {os.environ['GIT_COMMIT_MSG']} by {os.environ['COMMIT_AUTHOR_EMAIL']}\n")

# Example of a GitLab CI pipeline snippet
stages:
  - deploy
  - audit

deploy_stage:
  stage: deploy
  script:
    - echo "Deploying to production..."
    - git push origin master

audit_stage:
  stage: audit
  script:
    - cat /path/to/override_log.txt | sort -k2 | uniq -w64
```

**Example input / output:**

```text
# Example of an emergency commit message
$ git commit -m "Emergency fix for production issue #123"
```

**Flow:**

1. No
2. Yes
3. Emergency Pipeline Fix
4. Is it an emergency?
5. Log and Revert
6. Override Branch Protection
7. Audit Override After the Fact
8. Monthly Reconciliation Review

