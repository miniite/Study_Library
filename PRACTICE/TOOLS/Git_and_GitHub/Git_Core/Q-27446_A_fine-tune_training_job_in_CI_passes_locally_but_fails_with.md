---
type: "practice"
difficulty: "Deep"
---

# A fine-tune training job in CI passes locally but fails with 'dubious ownership' and missing-tag errors on the runner. Explain how CI's root-owned, shallow, detached-HEAD checkout causes each error and the specific fix.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Shallow Clones in CI](<../../../../TOOLS/Git_and_GitHub/Git_Core/Shallow_Clones_in_CI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we need to understand how Git checkouts work in CI environments and the implications of running a fine-tune training job on such a checkout. The errors "dubious ownership" and "missing-tag" are related to the way the Git repository is checked out and managed during the CI pipeline.

#### Triage

The question is essentially asking us to explain why a locally successful fine-tune training job might fail in a CI environment due to specific Git-related issues. The errors point towards problems with permissions and tag handling, which can be traced back to how the checkout process works in CI systems like GitHub Actions or Jenkins.

#### Assumptions
We are using a root-owned, shallow clone (or detached HEAD) of the Git repository.
The fine-tune training job involves fetching and tagging operations that might not work correctly on such checkouts.

#### Solution
from First Principles

Root-Owned Checkout:

In CI environments, it is common to use a root user for running jobs due to security reasons. This can lead to permission issues when the repository is checked out by the root user.
The dubious ownership error occurs because Git expects the working directory to be owned by the user who cloned the repository. When a CI runner uses the root user, it changes the ownership of the working directory to a non-existent UID (like 0), causing Git to flag this as dubious.

Shallow Clone:

A shallow clone only fetches a limited history of the repository (e.g., the last N commits). This can cause issues if the training job relies on specific tags or branches that are not present in the shallow clone.
The missing-tag error occurs because the tag is not available locally and Git cannot resolve it. Even though the tag might exist remotely, a shallow clone does not fetch all history by default.

Detached HEAD State:

When a CI job starts, it often checks out a specific commit or branch in a detached HEAD state. This means the working directory is at that specific point in history but not checked out from any particular branch.
The detached HEAD state can make it harder to perform operations like tagging and fetching full repository history.
Specific Fix

Change Checkout Method:

Use a full clone instead of a shallow clone. This ensures all branches and tags are available locally, resolving the missing-tag error.
BASH
git clone <repo-url> --depth 1000  # Adjust depth as needed

Check Ownership:

Ensure that the working directory is owned by a user with appropriate permissions before running the training job. This can be done using a chown command.
BASH
chown -R $USER:$GROUP .

Use Specific Branch or Tag:

If the fine-tune job requires specific branches or tags, explicitly checkout those first to ensure they are available in the working directory.
BASH
git checkout <branch-name>
git fetch --tags

Run as Non-Root User (if possible):

Running jobs as a non-root user with proper permissions can also resolve ownership issues.

#### Closing Summary
and Common Mistakes
The root-owned, shallow clone, and detached HEAD state are common pitfalls in CI environments.
Using a full clone ensures all necessary history and tags are available.
Changing ownership to the correct user is crucial for permission-related errors.
Explicitly checking out required branches or tags can avoid missing-tag issues.

#### Common Mistakes
Overlooking the need for a full clone, leading to missing tags.
Not changing file ownership before running jobs.
Assuming that a shallow clone will suffice for all operations involving branches and tags.

