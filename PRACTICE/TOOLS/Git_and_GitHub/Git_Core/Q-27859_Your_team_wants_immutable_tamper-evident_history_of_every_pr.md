---
type: "practice"
difficulty: "Deep"
---

# Your team wants immutable, tamper-evident history of every production schema change for compliance. Explain how signed commits, protected branches, and required linear history combine to give a defensible audit trail, and where the gaps still are.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Branch Protection Design](<../../../../TOOLS/Git_and_GitHub/Git_Core/Branch_Protection_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address your team's requirement for an immutable, tamper-evident history of every production schema change, we need to leverage Git and GitHub features effectively. By combining signed commits, protected branches, and required linear history, we can create a robust audit trail that is both defensible and reliable. However, there are still gaps in these protections that must be considered.

Firstly, let's break down the components:

Signed Commits: These ensure that every commit has a digital signature which verifies its authorship and integrity.
Protected Branches: These restrict direct pushes to specific branches, ensuring changes can only be made via pull requests (PRs) or merge commits.
Required Linear History: This ensures that each commit in the branch history follows sequentially from the previous one.

These features together provide a strong foundation for an audit trail. However, there are still some gaps and potential pitfalls we need to address:

#### Triaging the Question

The question is essentially asking how to ensure that every production schema change can be traced back reliably, with clear accountability and protection against unauthorized modifications. We need to consider both the technical setup and the operational practices required.

#### First-Principles Solution

Signed Commits:

Enable GPG signing in Git for all team members.
Automate commit signing where possible (e.g., via hooks or IDE plugins).

Protected Branches:

Protect the main branch to prevent direct pushes, ensuring that all changes go through PRs.
Use branches like staging and production for intermediate stages before merging into main.

Required Linear History:

Enable protected_branch settings in GitHub with required linear history turned on.
Detailed Implementation

Signed Commits:

BASH
git config user.signingkey your-gpg-key-id
git commit --gpg-sign=your-gpg-key-id -m "Commit message"

Protected Branches:

Go to the GitHub repository settings and navigate to Branches.
Select main branch, then click on Protect this branch.
Enable Allow force pushes only for trusted team members.
Add rules under Required status checks, ensuring all required actions pass before commits can be pushed.

Required Linear History:

JSON
{
  "branches": [
    {
      "name": "main",
      "protected": true,
      "required_linear_history": true
    }
  ]
}

#### Closing Summary
and Common Mistakes

While signed commits, protected branches, and required linear history provide a solid foundation, several gaps still exist:

Access Control: Ensure only authorized personnel have the ability to sign commits or make changes.
Backup Mechanisms: Have a robust backup strategy for schema changes in case of accidental deletions.
User Training: Educate team members on the importance and proper use of these features.
Automated Testing: Implement CI/CD pipelines with automated tests to catch issues early.

#### Common mistakes
include:

Overlooking GPG key management, which can lead to signing failures or unauthorized signatures.
Failing to update access controls as team structures change.
Not having a rollback plan in place for critical changes.

**Code:**

```bash
# Example of configuring GPG signing
git config user.signingkey your-gpg-key-id
git commit --gpg-sign=your-gpg-key-id -m "Commit message"
```

**Example input / output:**

```
Input: A developer pushes a commit to the main branch.
Output: The push fails, and a PR is required for the commit to be merged into main.
No
Yes
Developer Pushes Commit
Is main Protected?
Push Succeeds
PR Required
```

**Data:**

```json
{
  "branches": [
    {
      "name": "main",
      "protected": true,
      "required_linear_history": true
    }
  ]
}
```

