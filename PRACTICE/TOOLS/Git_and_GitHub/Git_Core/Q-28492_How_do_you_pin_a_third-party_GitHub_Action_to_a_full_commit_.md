---
type: "practice"
difficulty: "Deep"
---

# How do you pin a third-party GitHub Action to a full commit SHA, and why is that safer than pinning to a tag like v3?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Secure Workflows for Forks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Secure_Workflows_for_Forks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To pin a third-party GitHub Action to a specific commit SHA in your repository, you need to download the action code from the specified commit and check it into your own repository or an external storage location such as a Docker image registry. This approach is safer than pinning to a tag because using a commit SHA ensures that your workflow always uses exactly the same version of the action at a specific point in time, preventing accidental changes due to new releases.

#### Triage

The question is essentially asking how to ensure reproducibility and consistency when using third-party GitHub Actions by locking down to a precise code version. The interviewer might be probing whether you understand best practices for managing external dependencies and the implications of different ways to do so. Assumptions include that we have access to the required Git commands, Docker (if necessary), and familiarity with GitHub actions.

Step-by-Step Solution
Identify the Commit SHA: First, determine the exact commit SHA you want to pin your action to.
Download Action Code: Fetch the specific version of the action from its repository using git clone or by downloading it directly via an HTTP/HTTPS URL.
Check into Your Repository:
If using a Git-based approach, create a new branch in your repository and add the downloaded action files to this branch.
Alternatively, you can package the action as a Docker image if it's executable from one.

#### Example
Steps

Identify Commit SHA

SH
git rev-parse origin/<branch_name>@{upstream}

Download Action Code

SH
git clone --depth=1 -b <commit_sha> https://github.com/owner/repo.git /path/to/local/repo

Check into Your Repository

Create a new branch in your repository:
SH
cd /path/to/local/repo
git checkout -b my-action-branch
Add the action to your workflow file, specifying the local path:
YAML
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Use my custom action
        uses: /path/to/local/repo  # Adjust this to the actual local path
Why Pinning to a SHA is Safer Than Using Tags
Reproducibility: A specific commit SHA ensures that your workflow always runs with exactly the same version of the code, which is critical for consistent and repeatable builds.
Avoiding Breaking Changes: Newer tags or releases might introduce changes that break compatibility or add new features you don't need. By pinning to a specific commit, you avoid such issues.
Stability: Tags often represent major versions, while minor or patch versions can be more frequent and potentially unstable.

#### Closing Summary

Pinning a third-party GitHub Action to a full commit SHA is a safer practice than using tags because it ensures that your workflow uses exactly the same version of the action at all times. This approach provides reproducibility and stability by locking down specific code versions, preventing unexpected changes due to new releases or breaking changes.

#### Common Mistakes
Not Checking for Latest Issues: Always verify if there are any known issues with the commit SHA you plan to use.
Forgetting to Update Dependencies: Make sure your other dependencies are compatible with the pinned action version.
Neglecting to Test: After pinning, thoroughly test the workflow to ensure everything works as expected.
Final Self-check

This solution covers all aspects of pinning a third-party GitHub Action and explains why it is safer than using tags. It provides clear steps and addresses common pitfalls.

**Code:**

```
SH
Copy
git rev-parse origin/main@{upstream}
git clone --depth=1 -b <commit_sha> https://github.com/owner/repo.git /path/to/local/repo
cd /path/to/local/repo
git checkout -b my-action-branch
```

