---
type: "practice"
difficulty: "Intermediate"
---

# How would you enforce a minimum number of code reviewers before a merge that triggers deploy?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pull Request Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pull_Request_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce a minimum number of code reviewers before a merge that triggers deploy, we need to implement a CI/CD pipeline that includes a pre-deploy review step. This ensures that at least (N) developers have reviewed and approved the changes before deployment can proceed. The core idea is to couple the requirement for a minimum number of approvals with the deployment process.

The key questions this approach addresses include:

What is the minimum number of reviewers required?
How does the system determine if the required reviews are in place?
At what point in the CI/CD pipeline should the review step be enforced?

#### Triage

This question is probing our understanding of how to integrate code quality and governance into a continuous integration and deployment (CI/CD) process. The assumption here is that we have an existing CI/CD setup, possibly with Jenkins, GitLab CI, or GitHub Actions. We need to ensure that the merge trigger for deployments includes a review step before proceeding.

#### First-Principles Solution
Identify Merge Triggers: Determine where in your pipeline (e.g., pre-commit, post-merge) the code change triggers a deploy.
Review Requirement Check: Introduce a check to ensure at least (N) reviews are pending or completed before proceeding with the deployment.
Approval Mechanism: Implement a mechanism for tracking and validating the required number of reviewers.
Step-by-Step Implementation
Define the Minimum Reviewers: Set the minimum number of reviewers ((N)). For example, if (N = 2), then at least two reviews are needed.
Integrate with Code Repository: Use a version control system (VCS) like Git that supports pull requests and approvals. Ensure branches with merge triggers go through this process.
Pipeline Configuration: Configure the CI/CD pipeline to include a step before deployment checks if the required number of reviewers are present.

#### Example
Implementation

Let's use GitHub Actions as an example:

YAML
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy-checks:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Check for minimum reviews
        id: check_reviews
        run: |
          # Assuming a simple script to count reviews on the pull request
          review_count=$(gh pr checks --status=success | wc -l)
          if [ "$review_count" -lt 2 ]; then
            echo "::set-output name=reviews_needed::true"
          else
            echo "::set-output name=reviews_needed::false"
          fi

      - name: Deploy only if reviews are sufficient
        if: steps.check_reviews.outputs.reviews_needed == 'false'
        run: |
          # Deployment logic here, e.g., using kubectl or terraform

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By integrating a review step into the CI/CD pipeline, we ensure that code is reviewed before deployment, reducing potential issues in production.

#### Common Mistakes
Failing to integrate the review check with the merge trigger properly.
Not setting up a clear process for who can review and what constitutes an approved review.
Overlooking the need for communication between development and operations teams.

#### Follow-Ups
What if the reviews are not timely? How do we handle situations where reviewers are slow or unresponsive?
How does this affect developer productivity? Are there any ways to streamline the review process without compromising quality?

**Code:**

```yaml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy-checks:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Check for minimum reviews
        id: check_reviews
        run: |
          # Assuming a simple script to count reviews on the pull request
          review_count=$(gh pr checks --status=success | wc -l)
          if [ "$review_count" -lt 2 ]; then
            echo "::set-output name=reviews_needed::true"
          else
            echo "::set-output name=reviews_needed::false"
          fi

      - name: Deploy only if reviews are sufficient
        if: steps.check_reviews.outputs.reviews_needed == 'false'
        run: |
          # Deployment logic here, e.g., using kubectl or terraform
```

