---
type: "practice"
difficulty: "Deep"
---

# You want blue-green deploys where Git holds both color manifests and a switch, but a partial merge left the switch pointing at a color with no running pods. How would you make the color switch atomic in Git and validate it before the controller acts?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Protected Branches and Required Checks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Protected_Branches_and_Required_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To achieve an atomic blue-green deployment using Git, we need to ensure that the color switch is only updated when both the new manifest and the update to the switch itself are fully validated. This involves carefully managing the Git commit process, ensuring that changes to the manifests and the switch are treated as a single transaction.

#### Triaging the Question

The question revolves around handling partial merges in Git where a deployment script might leave the environment in an inconsistent state—specifically, with a color switch pointing to a manifest without any running pods. We need to make sure that any update to the manifests and their corresponding switches are validated together before being applied by our controller.

#### First-Principles Solution
Git Workflow: Ensure Git branches follow best practices such as feature branches, pull requests (PRs), and code reviews.
Atomic Commit: Treat changes to both the manifests and the switch as a single commit in Git to ensure they are either fully applied or not at all.
Pre-commit Validation: Run automated tests or checks before committing and pushing changes.
Post-Commit Validation: Validate the new state of the application before it is promoted by the controller.
Step-by-Step Solution

Create a New Feature Branch:

BASH
git checkout -b blue-green-update

Update Manifests and Switch:

Update the manifests for the new color.
Update the switch to point to the new manifest.

Pre-commit Validation:

Run automated tests on both the manifests and the updated switch.
BASH
make test-manifests
make test-switch

Commit Changes: If all validations pass, commit the changes with a single commit message to ensure atomicity:

BASH
git add .
git commit -m "Blue-green deploy: update manifests and switch"

Push Commit to Remote Repository: Ensure only validated commits are pushed:

BASH
git push origin blue-green-update

Merge Pull Request (PR): Open a PR on GitHub for the feature branch.

Reviewers should ensure that all changes are correct and validated.

Post-Commit Validation:

Manually or via a CI/CD pipeline, validate the new state by deploying to a staging environment if necessary.
BASH
make deploy-staging

Promote to Production (if successful): Once validated in staging, merge the PR and promote the changes to production:

BASH
git checkout master
git pull origin blue-green-update
git push origin master

#### Closing Summary
and Common Mistakes/Follow-Ups
Mistake 1: Committing changes without validation. Ensure all tests pass before committing.
Mistake 2: Failing to validate the new state after deployment. Always run post-deployment checks.

#### Follow-up
Consider using a CI/CD pipeline to automate the entire process, ensuring consistency and reducing human error.
Confirmation

This approach ensures that changes are atomic in Git and validated before being applied by the controller, preventing any partial deployments or inconsistent states.

