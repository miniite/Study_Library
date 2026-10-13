---
type: "practice"
difficulty: "Deep"
---

# A migration deploy from a Git tag ran in prod but the tag was later moved to a different commit, so you can't prove which SQL actually applied. How would you enforce tag immutability and verify the deploy resolves the tag to a fixed SHA before applying?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Tags and Releases](<../../../../TOOLS/Git_and_GitHub/Git_Core/Tags_and_Releases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of ensuring that Git tags remain immutable and verifying they map to a specific commit hash before deployment, we need to implement several best practices and tools in our workflow. This involves understanding how Git tags work, setting up proper tag management policies, and integrating these with GitHub's features.

TL;DR: We will enforce tag immutability by setting up an automated process that ensures the tag points to a specific commit hash before deployment. This can be achieved using GitHub Actions for validation checks and Git hooks for local enforcement.

The core of this question revolves around ensuring the integrity and immutability of Git tags, particularly in a production environment where deployments are critical. The interviewer is probing your understanding of Git tag management, automated deployment processes, and how to prevent accidental changes or misuse of tags. Key assumptions include that we are using GitHub for version control and that we have access to its features for automation.

Understand Tags in Git: A Git tag is a reference to a specific commit hash, similar to a bookmark. Unlike branches, which can be moved or deleted, tags should ideally remain immutable once created.
Automated Validation with GitHub Actions: We need an automated process that checks if the tag points to a specific commit before allowing any deployment.
Local Enforcement with Git Hooks: To prevent accidental changes, we will use pre-commit hooks on developers' machines to ensure they do not modify existing tags.
Step-by-Step Solution

Create a Protected Tag Policy in GitHub:

Go to the repository settings in GitHub and set up protected branches and tags.
Configure the tag protection policy to disallow forced pushes, deletions, or renaming of specific tags used for deployments.

GitHub Actions Workflow:

Create a GitHub Action that runs on push events to a specified branch (e.g., main).
This action will verify if any tags created or modified point to the correct commit hash.
YAML
name: Validate Tag Mappings

on:
  push:
    branches:
      - main

jobs:
  validate-tags:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2

      - name: Verify Tag References
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          # Check if new tags are created or modified in the push event.
          git diff --name-only --cached | grep -E 'refs/tags/.+' > new_tags.txt

          for tag in $(cat new_tags.txt); do
            sha=$(git rev-parse "$tag")
            echo "::set-output name=tag_sha::$sha"
            # Compare SHA with expected commit hash from the deployment pipeline.
            if [ "$sha" != "expected_commit_hash" ]; then
              echo "Tag '$tag' points to an unexpected commit. Deployment aborted."
              exit 1
            fi
          done

Pre-Commit Hooks for Local Environments:

Implement pre-commit hooks that enforce the use of immutable tags.
BASH
# .git/hooks/pre-commit

#!/bin/bash

git diff --cached --name-only | grep -E 'refs/tags/.+' > new_tags.txt

while read -r tag; do sha=$(git rev-parse "$tag") if [ "$sha" != "expected_commit_hash" ]; then echo "Tag '$tag' points to an unexpected commit. Tag modification not allowed." exit 1 fi done < new_tags.txt

CODE

4. **Ensure Developers Use Correct Tags**:
- Provide clear documentation and training for developers on how to use tags correctly.
- Integrate tag usage checks into CI/CD pipelines.

### Closing Summary + Common Mistakes/Follow-Ups

By implementing these steps, we ensure that Git tags remain immutable and can be confidently used in deployments. The key is to leverage GitHub Actions for automated validation and pre-commit hooks for local enforcement. Common mistakes include neglecting local enforcement, failing to document tag usage policies, or not setting up protected tag policies in GitHub.

If the interviewer asks follow-up questions, you could discuss:
- How to handle edge cases where tags need to be modified (e.g., rollback scenarios).
- Integrating with other CI/CD tools like Jenkins or GitLab.
- Security considerations around exposing commit hashes.

**Comparison:**

```
Tag	Expected SHA	Actual SHA
v1.0	123abc	123abc
v1.1	456def	789ghi

If v1.1 points to 789ghi, the deployment would fail.
```

**Code:**

```

name: Validate Tag Mappings

on: push: branches: - main

jobs: validate-tags: runs-on: ubuntu-latest

CODE
Copy
steps:
  - name: Checkout Repository
    uses: actions/checkout@v2

  - name: Verify Tag References
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    run: |
      git diff --name-only --cached | grep -E 'refs/tags/.+' > new_tags.txt

      for tag in $(cat new_tags.txt); do
        sha=$(git rev-parse "$tag")
        echo "::set-output name=tag_sha::$sha"
        if [ "$sha" != "expected_commit_hash" ]; then
          echo "Tag '$tag' points to an unexpected commit. Deployment aborted."
          exit 1
        fi
      done
CODE
Copy

```bash
#!/bin/bash

git diff --cached --name-only | grep -E 'refs/tags/.+' > new_tags.txt

while read -r tag; do
  sha=$(git rev-parse "$tag")
  if [ "$sha" != "expected_commit_hash" ]; then
    echo "Tag '$tag' points to an unexpected commit. Tag modification not allowed."
    exit 1
  fi
done < new_tags.txt
```

**Example input / output:**

```text
Input:
- Push to branch `main` with tag `v1.0`

Output:
- GitHub Action runs and checks if `v1.0` points to the correct commit hash.
- Pre-commit hook on developer's machine ensures no changes are made to immutable tags.

Dry-run:
- Tag `v1.0` is pushed and checked against expected SHA.
- If `expected_commit_hash` does not match, deployment aborts with an error message.
```

**Data:**

```json
{
  "new_tags": [
    {
      "name": "v1.0",
      "commit_sha": "expected_commit_hash"
    },
    {
      "name": "v1.1",
      "commit_sha": "unexpected_commit_hash"
    }
  ]
}
```

**Flow:**

1. Yes
2. Success
3. Failure
4. No
5. Valid Tag
6. Invalid Tag
7. Push to main branch
8. Is new tag created ormodified?
9. Run GitHub Action
10. Check tag against expectedSHA
11. Deployment allowed
12. Abort deployment
13. Run pre-commit hook on localmachine

