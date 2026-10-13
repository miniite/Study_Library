---
type: "practice"
difficulty: "Intermediate"
---

# Your team wants git blame on a stored-procedure file to point to the real author of a logic bug, but a mass reformat (sqlfluff fix) poisoned blame across the file. How would you set up .git-blame-ignore-revs and what are its limits during a live incident?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Blame and Ignore Revisions](<../../../../TOOLS/Git_and_GitHub/Git_Core/Blame_and_Ignore_Revisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we need to understand that git blame relies on line-by-line annotations of who last modified each line of code. However, when files are re-formatted or changed in bulk, these annotations can become inaccurate. In your scenario, a mass reformat using sqlfluff fix has likely caused git blame to misattribute changes. To correct this, you'll need to set up .git-blame-ignore-revs. Here’s how to do it and the limitations involved.

Firstly, understand that .git-blame-ignore-revs is a file-level setting in Git that tells Git not to track blame for specific revisions on a particular file. This can help isolate changes caused by the reformat from genuine code modifications.

#### Triage
What’s Being Probed

The interviewer is probing your understanding of how git blame works, the implications of mass file reformats, and how to mitigate such issues using Git tools. You will need to consider assumptions about the team's practices, the version control history, and the specific context in which this reformat happened.

#### Assumptions
The team is familiar with Git commands and workflows.
The stored-procedure file has a significant number of lines (enough for sqlfluff to have an impact).
There are no other recent changes that complicate the blame annotations.
Step-by-Step Solution

Identify Affected Revisions: Determine which revisions were part of the bulk reformat process. For example, if a commit message clearly states "Bulk reformat using sqlfluff fix," you can identify this revision range easily.

Create .git-blame-ignore-revs: Create or update the .git-blame-ignore-revs file in your repository to include the revisions associated with the bulk reformat.

BASH
echo "473a658b" >> .git-blame-ignore-revs  # Replace with the actual commit SHA of the reformat

This tells Git not to blame these specific lines for changes made by this commit.

Apply git blame Again: Run git blame again after setting up .git-blame-ignore-revs to see if blame annotations now make more sense.

BASH
git blame --ignore-revs-file=.git-blame-ignore-revs -e your_stored_procedure.sql

Monitor and Validate: After applying the ignore rules, closely monitor how git blame behaves. Ensure that it correctly identifies changes made by actual contributors rather than those caused by the reformat.

#### Closing Summary

While .git-blame-ignore-revs can help clean up blame annotations after a bulk reformat, it has several limitations and should be used cautiously:

It only affects blame annotations on a file-level basis.
Misconfigurations or incorrect revisions in the ignore list can still cause issues.
Frequent bulk reformats might require ongoing maintenance of the ignore list.

#### Common Mistakes/Follow-Ups
Ignoring Too Much: Overusing .git-blame-ignore-revs could lead to ignoring legitimate blame changes, making it harder to track down real contributions.
Incomplete List: Missing revisions in the ignore-revs-file can result in still-misattributed blame.
Team-wide Practice: Consider setting up a team practice for avoiding bulk reformats where possible and ensuring clear commit messages for such actions.

**Code:**

```bash
echo "473a658b" >> .git-blame-ignore-revs  # Example of adding a revision to ignore
git blame --ignore-revs-file=.git-blame-ignore-revs -e your_stored_procedure.sql
```

