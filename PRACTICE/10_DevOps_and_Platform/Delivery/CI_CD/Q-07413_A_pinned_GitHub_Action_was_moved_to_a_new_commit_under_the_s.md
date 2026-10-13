---
type: "practice"
difficulty: "Intermediate"
---

# A pinned GitHub Action was moved to a new commit under the same tag and silently changed behavior mid-pipeline; how would you pin by SHA across dozens of pipelines and detect when a pinned action drifts?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Build and Release Metadata](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Build_and_Release_Metadata/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a pinned GitHub Action remains consistent across multiple pipelines, we need to transition from using tags to using SHAs in our package.json or .github/workflows/* files. This approach will help us detect when the action's behavior changes unexpectedly. Here’s how you can achieve this:

Understand the Problem: The issue lies in using a tag that points to an outdated version of the GitHub Action, which got inadvertently updated mid-pipeline execution.

Transition Strategy: We need to shift from pinning by tags (e.g., @latest, v0.9.3) to pinning by SHAs (commit hashes) directly in our configuration files. This ensures that we always use a specific commit, avoiding the risk of mid-pipeline changes.

Triaging and Assumptions
We are dealing with multiple pipelines triggered across different repositories.
The change needs to be implemented for both existing and new pipelines.
Monitoring tools like GitHub Actions' built-in alerts or third-party monitoring services will help detect any drifts in the pinned action's behavior.

#### First Principles Solution
Identify Existing Pipelines: List all pipelines that use the problematic tag.
Update Configuration Files:
Locate and update package.json or .github/workflows/* files to use SHAs instead of tags.
Implement SHA-based Pinning:
JSON
// Example: Updating package.json in a Node.js project
"dependencies": {
  "my-action": "<SHA>"
}
GitHub Actions Workflow Update: Ensure the .github/workflows/* files reference the correct SHAs.
Automated Verification:
Use GitHub’s built-in or external tools to verify that all pipelines are using the intended SHAs.
Monitoring and Alerting:
Set up alerts for any drifts in action behavior by monitoring logs, artifact versions, and build outputs.

#### Implementation
Steps

Identify Old Pipelines:

BASH
# Example command to find all pipelines that use the old tag
git grep -l 'v0.9.3' .github/workflows/*.yml

Update Configurations:

JSON
// Before: package.json
"dependencies": {
  "my-action": "v0.9.3"
}

// After: package.json
"dependencies": {
  "my-action": "<SHA>"
}

GitHub Actions Workflow Update:

YAML
# Before using a tag
- uses: actions/stale@v1
  with:
    repository: my-repo
    pr-labels: stale
    tag-filter: v0.9.3

# After using a SHA
- uses: actions/stale@v1
  with:
    repository: my-repo
    pr-labels: stale
    sha: <SHA>

Verification and Monitoring:

BASH
# Example script to verify SHAs in workflows
find .github/workflows -type f -exec grep -H 'v0.9.3' {} \; > outdated-workflows.txt

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Continuing to use tags instead of SHAs, leading to drifts in action behavior.
Not thoroughly testing the updated configurations before deploying them.

#### Follow-Up
Questions:

How would you handle actions with multiple authors or contributors?
What are the implications for versioning and change management processes?

**Code:**

```json
// Example package.json update
{
  "dependencies": {
    "my-action": "<SHA>"
  }
}
```

**Example input / output:**

```text
# Example output of finding old pipelines
./.github/workflows/pipeline1.yml:uses: actions/stale@v1 with { repository: my-repo, pr-labels: stale, tag-filter: v0.9.3 }
./.github/workflows/pipeline2.yml:uses: actions/stale@v1 with { repository: my-repo, pr-labels: stale, tag-filter: v0.9.3 }
```

**Flow:**

1. Identify Pipelines
2. Update Configurations
3. Check for SHA Consistency
4. Monitor and Alert
5. Confirm Stability

