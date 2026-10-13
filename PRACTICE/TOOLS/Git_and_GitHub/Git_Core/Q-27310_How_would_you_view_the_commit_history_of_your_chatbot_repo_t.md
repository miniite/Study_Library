---
type: "practice"
difficulty: "Basic"
---

# How would you view the commit history of your chatbot repo to remember what prompt changes you made last week?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Searching History](<../../../../TOOLS/Git_and_GitHub/Git_Core/Searching_History.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To recall what prompt changes were made in my chatbot repository from last week, I would leverage Git and GitHub's rich feature set to view the commit history. This approach involves several steps, including navigating through branches and tags, using git log with specific filters, and possibly collaborating with version control tools.

#### Triage

The question is essentially asking how to use Git commands effectively to find a specific commit in your chatbot repository that corresponds to changes made last week. The key here is understanding the timeline of commits and filtering them based on date.

Step-by-Step Solution From First Principles
Identify the Time Frame: Determine the exact time frame you need to search for, which is "last week."
Navigate Through Commits:
Use git log with date filters.
Optionally use branches or tags if specific versions are relevant.

#### Example
Workflow
1. Checking Recent Commit History

First, I would check the recent commits to see if last week's changes stand out:

BASH
git log --oneline --since="last week" --prompt

This command lists all commits that happened in the past week and includes a reference to prompt files.

2. Filtering by Date

If I want more precise control over the date range, I can use the --until and --after options:

BASH
git log --oneline --since="2023-10-05" --until="2023-10-12" --prompt

Replace the dates with actual values to match last week's exact timeline.

3. Using Branches or Tags

If specific branches or tags are relevant, you can filter by them:

BASH
git log --oneline --since="last week" --branch=feature/prompt-updates --prompt

This narrows down the search to commits on a particular branch.

Full Solution Summary

By combining git log with appropriate date filters and considering branches or tags, you can pinpoint the exact prompt changes made last week. This method ensures that you capture all relevant information without overwhelming yourself with too many irrelevant commits.

#### Common Mistakes
and Follow-Ups
Not Using Date Filters: Failing to use date filters might result in a lot of noise, making it hard to find specific commits.
Overlooking Branches/Tags: Assuming the history is linear can lead you to miss important changes if they were made on a different branch or tagged version.

By carefully selecting the right commands and parameters, you can efficiently manage your repository's commit history and stay organized.

