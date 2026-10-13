---
type: "practice"
difficulty: "Deep"
---

# You maintain a shared Ansible-roles repo consumed by 20 playbook repos via a Git requirements file. A breaking role change went out because consumers tracked the main branch. Design a versioning and pinning scheme so a role change can't silently reach every consumer.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reusable_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a role change does not silently propagate to all playbook repos, we need to design a robust versioning and pinning strategy using Git tags and GitHub Actions. The goal is to maintain transparency and control over dependencies between repositories.

#### Triaging the Question

The core issue here is ensuring that a breaking change in an Ansible role doesn't silently affect multiple dependent projects by tracking the main branch. We need a system that enforces versioning, pinning, and review before a role update reaches all consumers. This involves using Git tags to label specific versions of roles, leveraging GitHub Actions for automated checks, and implementing a workflow where updates go through a review process.

Step-by-Step Solution

Version Tagging: Each release of an Ansible role should be tagged with semantic versioning (e.g., v0.12.3). This ensures that roles are clearly labeled and can be referenced precisely.

Dependency Pinning in Requirements File:

#### Cons
umers of the Ansible roles should pin to specific tags in their requirements file, not track branches directly. For example, /path/to/requirements.yml could contain role:username/role-name==v0.12.3.

GitHub Actions for Automated Checks:

Create a GitHub Action that runs before pushing changes to the main branch in the Ansible roles repository.
The action should validate that:
New versions of roles are tagged with semantic versioning.
Changes do not introduce breaking changes (e.g., by using tools like ansible-lint).
Pull requests are reviewed before merging.

Release Workflow:

When a new role version is required, create a pull request to update the role and tag it appropriately (git tag v0.12.4).
Merge the pull request after approval, but only after confirming that all relevant checks pass.
Push the tagged commit to main, triggering an automated release process.

Updating Consumers:

When a new version is released, consumers should update their requirements files manually or via automated scripts to reference the latest stable tag (e.g., v0.12.4).

#### Closing Summary
and Common Mistakes

By implementing this strategy, we ensure that role updates are managed transparently, with clear tagging, review, and approval processes. The key is to avoid tracking branches directly in requirements files and instead rely on semantic versioning tags.

#### Common Mistakes
Not using Git Tags: Failing to use Git tags for versioning can lead to confusion and issues.
Frequent Branch Merging: Directly merging to main without thorough testing or review cycles can introduce bugs.
Lack of Automated Checks: Not leveraging GitHub Actions for automated validation can delay issue detection.

#### Example
Workflow

Role Development Repository (Ansible Roles):

Pull Request: Update role code
Action Run: Validation checks pass
Tagging: git tag v0.12.4 && git push --tags
Merge: After approval, commit tagged version to main

Playbook Repository:

Update requirements.yml: role:username/role-name==v0.12.4
Action Run: Validate role is properly installed and functional
YAML
# Example requirements file snippet
- name: username.role_name
  src: git+https://github.com/username/role_name.git#v0.12.4
Self-check

This solution ensures robust versioning, pinning, and automated checks are in place to prevent silent role changes.

**Code:**

```yaml
# Example GitHub Action for validating releases
on:
  push:
    branches:
      - main

jobs:
  validate_release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Validate semantic versioning
        run: |
          # Check for semantic version format
          if ! [[ $GITHUB_REF =~ v[0-9]+\.[0-9]+\.[0-9]+ ]]; then
            echo "Version must be in semver format (vX.Y.Z)"
            exit 1
          fi
      - name: Run ansible-lint
        run: |
          # Run linter to check for breaking changes
          ansible-lint
```

