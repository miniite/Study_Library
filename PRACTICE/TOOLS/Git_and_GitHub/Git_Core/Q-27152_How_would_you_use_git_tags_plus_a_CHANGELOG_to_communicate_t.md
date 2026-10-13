---
type: "practice"
difficulty: "Intermediate"
---

# How would you use git tags plus a CHANGELOG to communicate to your team which model version fixed which data issue?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Tags and Releases](<../../../../TOOLS/Git_and_GitHub/Git_Core/Tags_and_Releases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, as an ML engineer with intermediate experience, using Git tags and a CHANGELOG is crucial for maintaining clear communication about model versions and their associated fixes. The core idea here is to leverage version control effectively so that your team can easily track which specific codebase resolved particular issues in the data.

Firstly, let's clarify what we're trying to achieve:

Git Tags: These are used to mark significant points in a repository’s history.
CHANGELOG: A file that logs changes made in each version of the project. It typically includes information on new features, bug fixes, and breaking changes.

Now, step-by-step from first principles:

Identify the Issue:

When a data issue is identified, document it clearly in your issue tracking system (e.g., Jira, GitLab).
This document should include details like the timestamp, description of the problem, and any relevant metadata.

Create a New Feature Branch:

Once an issue is identified, create a new feature branch from the latest main or master branch.
BASH
git checkout -b fix-issue-123

Implement the Fix:

Develop and test the changes to resolve the data issue.
Ensure all tests pass before committing.

Commit and Push Changes:

Commit your changes with a descriptive message, including references to the Jira ticket.
BASH
git commit -am "Fix #123: Correct erroneous data handling"
Push the changes to your remote repository.

Update CHANGELOG:

Add an entry in the CHANGELOG.md file under the latest version's section (or create a new one if necessary).
MARKDOWN
## [Version 1.0.4] - 2023-10-06

### Fixed
- #123: Correct erroneous data handling

Tag the Version:

Once the issue is resolved and tested, create a tag for this version.
BASH
git tag v1.0.4
git push origin v1.0.4

Merge Back to Main Branch (Optional):

If the changes are stable and ready, merge them back into main.
BASH
git checkout main
git pull origin main
git merge fix-issue-123 --no-ff  # Non-fast-forward merge
git push origin main

Clean Up:

Delete the feature branch once it's no longer needed.
BASH
git branch -d fix-issue-123

By following these steps, you ensure that your team can quickly understand which model version addressed a particular issue by simply looking at tags and CHANGELOG entries.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

This approach provides transparency and traceability in the development process. However, common pitfalls include forgetting to update the CHANGELOG or tagging the wrong commit. Always ensure that every fix is properly documented and tagged to avoid confusion later.

**Code:**

```bash
# Example Git Commands
git checkout -b fix-issue-123
git commit -am "Fix #123: Correct erroneous data handling"
git tag v1.0.4
git push origin v1.0.4
```

**Flow:**

1. Identify Issue
2. Create Feature Branch
3. Implement Fix & Test
4. Commit and Push Changes
5. Update CHANGELOG
6. Tag Version
7. Merge to Main (Optional)
8. Clean Up

