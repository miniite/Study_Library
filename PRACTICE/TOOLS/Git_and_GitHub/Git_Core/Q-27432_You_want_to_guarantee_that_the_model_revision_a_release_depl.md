---
type: "practice"
difficulty: "Deep"
---

# You want to guarantee that the model revision a release deploys from is immutable so a re-tag can't silently change what's in prod. How would you enforce tag immutability at the GitHub repo level and verify the pin resolves to a fixed SHA?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Tags and Releases](<../../../../TOOLS/Git_and_GitHub/Git_Core/Tags_and_Releases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that model revisions deployed from a Git repository are immutable, we need to prevent any re-tagging or changes to what's already tagged in production. We'll enforce this immutability at the GitHub repo level by leveraging Git tags and ensuring that these tags resolve to fixed SHAs. Here’s how you can implement this:

Understand What You’re Probing: The interviewer is asking about how to manage and verify the immutability of tags in a Git repository, specifically within a GitHub environment. They are probing your understanding of version control practices, Git operations, and security measures.

State Assumptions:

The deployment process relies on Git tags for identifying specific versions.
You have appropriate permissions to manage the repository.
There is an existing CI/CD pipeline that automatically deploys based on these tags.

#### First Principles Solution

Tagging and Commit SHAs: Ensure that each tag points to a unique commit SHA.
Tag Protection: Disable force pushes for tags, so they cannot be changed or overwritten accidentally.
Automated Verification: Use GitHub Actions to verify that the tag indeed resolves to the expected SHA before deployment.
Step-by-Step Solution

Disabling Force Pushes on Tags:

Open your repository settings in GitHub and navigate to "Settings" -> "Danger zone".
Click on “Disable force push” for tags.
This ensures that no one can accidentally (or maliciously) change the commit associated with a tag.

Creating a Tag Protection Policy:

In the same settings, go to “Tagging restrictions” under “Danger zone”.
Define a policy where you specify which users or teams are allowed to create or update tags. This can be useful for controlling who has the authority to make such changes.

Automating Tag Verification with GitHub Actions:

Create a GitHub Action that runs automatically when new tags are created.
The action should verify the SHAs of each tag against your production environment’s known good SHAs.
Here is an example workflow:
YAML
name: Verify Tag

on:
  push:
    tags:
      - 'v*'

jobs:
  verify-tag:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: 3.8

      - name: Install dependencies
        run: |
          pip install requests gitpython

      - name: Get tag list from GitHub API
        id: get_tags
        run: |
          token=$(cat .github_token)
          tags=$(curl -H "Authorization: token $token" https://api.github.com/repos/owner/repo/tags | jq -r '.[].name')
          echo "tags=$tags" >> $GITHUB_OUTPUT

      - name: Verify tags against known SHAs
        run: |
          python verify_tags.py

Implementing the Verification Script:

PYTHON
import requests
from git import Repo, GitCommandError

# Load your GitHub token and the production environment's SHAs
with open(".github_token") as f:
    token = f.read().strip()
known_good_shas = {
    "v1.0": "abc123",
    "v1.1": "def456"
}

headers = {"Authorization": f"token {token}"}

def check_tag(tag, sha):
    url = f"https://api.github.com/repos/owner/repo/git/ref/tags/{tag}"
    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        ref_info = response.json()
        target_sha = ref_info["object"]["sha"]
        return target_sha == sha
    else:
        print(f"Failed to fetch tag {tag}: {response.text}")
        return False

# Verify each known good tag
for tag, expected_sha in known_good_shas.items():
    if not check_tag(tag, expected_sha):
        raise ValueError(f"Tag {tag} does not point to the correct SHA: {expected_sha}")

print("All tags verified successfully.")

#### Closing Summary
and Common Mistakes/Follow-Ups:

Ensure that all deployed models are tagged with meaningful names.
Regularly update your known good SHAs list in your verification script.
Monitor the logs of GitHub Actions to catch any issues early.
Closing Thoughts
This approach ensures that only authorized personnel can create tags, and these tags cannot be altered once created. By automating tag verification using GitHub Actions, you can prevent silent changes to what's in production.

#### Common mistakes
include not having strict tagging policies or failing to automate the verification process, leading to potential misconfigurations.

**Comparison:**

```
Step	Description
1	Disable force pushes for tags in GitHub settings.
2	Define tagging restrictions policy.
3	Create a GitHub Action for automatic tag verification.
```

**Code:**

```yaml
name: Verify Tag

on:
  push:
    tags:
      - 'v*'

jobs:
  verify-tag:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: 3.8

      - name: Install dependencies
        run: |
          pip install requests gitpython

      - name: Get tag list from GitHub API
        id: get_tags
        run: |
          token=$(cat .github_token)
          tags=$(curl -H "Authorization: token $token" https://api.github.com/repos/owner/repo/tags | jq -r '.[].name')
          echo "tags=$tags" >> $GITHUB_OUTPUT

      - name: Verify tags against known SHAs
        run: |
          python verify_tags.py
```

**Example input / output:**

```text
# Example input and output for the verification script

import requests

token = "your-github-token-here"
headers = {"Authorization": f"token {token}"}

tag_name = "v1.0"
url = f"https://api.github.com/repos/owner/repo/git/ref/tags/{tag_name}"
response = requests.get(url, headers=headers)

if response.status_code == 200:
    ref_info = response.json()
    target_sha = ref_info["object"]["sha"]
    print(f"Tag {tag_name} points to: {target_sha}")
else:
    print(f"Failed to fetch tag {tag_name}: {response.text}")

# Output example
# Tag v1.0 points to: abc123
```

**Data:**

```yaml
known_good_shas = {
    "v1.0": "abc123",
    "v1.1": "def456"
}
```

**Flow:**

1. No
2. Yes
3. Tag pushed to repository
4. Is the tag protected?
5. Force push is allowed, tag canbe changed
6. Tag verification starts withGitHub Action
7. Verify SHAs of tags againstknown good SHAs

