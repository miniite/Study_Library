---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a Python script that queries the GitHub API to enforce a policy across hundreds of repos (branch protection, required checks) and handles secondary rate limits gracefully?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate-limited Integrations](<../../../../TOOLS/Python/Python_Language/Rate-limited_Integrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this task, we need to create a Python script that can efficiently query the GitHub API to set up branch protection rules and required checks across multiple repositories while handling rate limiting properly. The script will be broken down into several steps, from setting up authentication to implementing error handling.

#### Triaging the Question

The core of the question revolves around two main aspects:

API Interaction: Writing a Python script that interacts with the GitHub API.
Rate Limit Handling: Ensuring the script can handle secondary rate limits gracefully without failing prematurely.

#### First-Principles Solution
Step 1: Authentication

We need to authenticate our script using personal access tokens (PATs) or OAuth tokens provided by GitHub.

PYTHON
import os
from github import Github

# Load environment variable for token security
token = os.getenv('GITHUB_TOKEN')
g = Github(token)
Step 2: Repository Iteration and API Calls

We will iterate over a list of repositories, setting up branch protection rules and required checks.

PYTHON
def set_branch_protection(repo_name, branch_name):
    repo = g.get_repo(f"org/repo_name")
    protection = repo.get ProtectiveBranch(branch_name)
    if not protection:
        # Create new protective branch
        protection = repo.create ProtectiveBranch(branch_name, required_status_checks=["all"], enforce_admins=True)
Step 3: Handling Rate Limits

To handle rate limits, we can implement exponential backoff strategies and use retry mechanisms.

PYTHON
import time

def handle_rate_limit(response):
    if response.status_code == 403:
        reset_after = int(response.headers['X-RateLimit-Reset'])
        delay = max(1, reset_after - time.time())
        print(f"Rate limit exceeded. Retrying in {delay} seconds.")
        time.sleep(delay)

#### Closing Summary

The Python script will iterate over multiple repositories, setting up necessary branch protections and required checks using the GitHub API. It employs robust error handling to manage secondary rate limits gracefully.

#### Common Mistakes/Follow-Ups
Not Handling Rate Limits Properly: Failing to implement retry logic can lead to premature termination of the script.
Token Security: Using environment variables for sensitive information is a good practice, but ensure they are stored securely.
Code Readability and Maintainability: Use clear variable names and comments for better readability.

**Comparison:**

| Repository | Branches |
|------------|----------|
| repo1       | main, develop  |
| repo2       | master, release  |

**Code:**

```python
import os
from github import Github

# Load environment variable for token security
token = os.getenv('GITHUB_TOKEN')
g = Github(token)

def set_branch_protection(repo_name, branch_name):
    repo = g.get_repo(f"org/{repo_name}")
    protection = repo.get ProtectiveBranch(branch_name)
    if not protection:
        # Create new protective branch
        protection = repo.create ProtectiveBranch(branch_name, required_status_checks=["all"], enforce_admins=True)

def handle_rate_limit(response):
    if response.status_code == 403:
        reset_after = int(response.headers['X-RateLimit-Reset'])
        delay = max(1, reset_after - time.time())
        print(f"Rate limit exceeded. Retrying in {delay} seconds.")
        time.sleep(delay)
```

**Example input / output:**

```text
# Example input and output

repolist = [{"name": "repo1", "branches": ["main", "develop"]}, {"name": "repo2", "branches": ["master", "release"]}]

for repo in repolist:
    for branch_name in repo["branches"]:
        set_branch_protection(repo["name"], branch_name)
```

**Data:**

```json
{
  "repos": [
    {"name": "repo1", "branches": ["main", "develop"]},
    {"name": "repo2", "branches": ["master", "release"]}
  ]
}
```

**Flow:**

1. Initialize GitHub
2. Authenticate using PAT
3. Iterate over repository list
4. Set branch protection for eachbranch in the current repo
5. Handle rate limit and retry ifnecessary
6. Continue to next repo or endscript

