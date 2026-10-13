---
type: "practice"
difficulty: "Intermediate"
---

# How would you configure branch protection rules on GitHub to require CI to pass before anyone can merge to main?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Signed Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Signed_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that only changes passing CI are merged into the main branch, we need to set up branch protection rules in GitHub. This involves configuring the settings on the repository to enforce mandatory checks before allowing a pull request (PR) to be accepted. Here’s how you can do it:

First, let's understand what branch protection rules are and why they are necessary. These rules allow you to control access to branches by specifying which teams or users can push to specific branches and whether certain activities, like merging into protected branches, require approvals.

To proceed with the setup, we will follow these steps:

Log in to GitHub and navigate to your repository.
Go to the settings tab of the repository.
Scroll down to find the "Branches" section.
Click on the "main" branch (or any other protected branch) to configure its protection rules.

Now, let's dive into configuring the CI requirement:

**Comparison:**

| Setting           | Value                        |
|-------------------|------------------------------|
| Branch Name       | main                         |
| Required Status Checks (Strict) | true                     |
| Contexts          | continuous-integration/circleci, continuous-integration/travis-ci  |
| Enforce Admins    | false                        |
| Required Reviews  | None                         |
| Restrictions      | None                         |

**Code:**

```plaintext
# Step 1: Navigate to GitHub and select your repository settings

# Step 2: Go to the "Branches" section in the settings tab

# Step 3: Select the main branch for protection rules
```

**Example input / output:**

```text
# Input (Step-by-step process)
1. Login to GitHub and go to the repository settings.
2. Click on "Branches" in the settings tab.
3. Select the main branch for protection rules.

# Output (Configured settings)
{
  "branch_name": "main",
  "required_status_checks": {
    "strict": true,
    "contexts": ["continuous-integration/circleci", "continuous-integration/travis-ci"]
  },
  "enforce_admins": false,
  "required_pull_request_reviews": null,
  "restrictions": null
}
```

**Data:**

```json
{
  "branch_name": "main",
  "required_status_checks": {
    "strict": true,
    "contexts": ["continuous-integration/circleci", "continuous-integration/travis-ci"]
  },
  "enforce_admins": false,
  "required_pull_request_reviews": null,
  "restrictions": null
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Log in to GitHub and select repository settings] --> B[Navigate to "Branches" section];
4. B --> C[Select "main" branch for protection rules];
5. C --> D[Configure required status checks to pass CI];
6. D --> E[Save changes];

