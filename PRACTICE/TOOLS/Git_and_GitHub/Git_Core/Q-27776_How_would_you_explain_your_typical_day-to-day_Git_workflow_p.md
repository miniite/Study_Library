---
type: "practice"
difficulty: "Intermediate"
---

# How would you explain your typical day-to-day Git workflow (pull, branch, write migration, commit, PR, merge) on a database team during an interview?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Branching Strategies](<../../../../TOOLS/Git_and_GitHub/Git_Core/Branching_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In my daily routine as a database developer, Git and GitHub form the backbone of our collaborative development process. A typical day involves working with branches, writing migrations, committing changes, and managing pull requests (PRs) to ensure smooth integration into our main codebase.

First, I start by checking out a new branch for any feature or bug fix using git checkout -b <branch-name>. This allows me to work on isolated tasks without affecting the master branch. Once I've made changes, particularly when dealing with database schema modifications, I use SQL migrations to version control these changes.

Next, I commit my changes along with a descriptive message such as "Refactor users table for performance improvements." Commit messages are crucial because they provide context and history of changes, making future maintenance easier. After committing, I push the branch to GitHub using git push origin <branch-name> so that my teammates can review it.

To integrate these changes into our main project, I create a pull request (PR) on GitHub. The PR template requires me to describe the change, link any relevant issues or tickets, and provide detailed steps on how to test the new feature. My teammates then review the code and migrations for correctness, performance, and adherence to coding standards.

If the PR is approved, it undergoes a merge process where I rebase my branch onto the latest changes from the main branch using git pull --rebase origin main. This ensures that conflicts are resolved early, making merges smoother. Once rebased, I force push back to GitHub with git push -f origin <branch-name>.

Finally, if no issues arise during testing or review, I merge the PR into the main branch using git merge <branch-name>, followed by a clean-up step where I delete the now-merged branch from both my local and remote repositories. This keeps our repository tidy and easy to navigate.

**Code:**

```
SH
Copy
# Example commands for Git workflow
git checkout -b feature-abc
# Making changes and writing SQL migrations
git commit -m "Refactor users table for performance improvements"
git push origin feature-abc
```

**Flow:**

1. Yes
2. Yes
3. Yes
4. No
5. Start
6. Check out a new branch?
7. Merge conflicts may occur
8. Write SQL migrations, commitchanges
9. Push to GitHub (origin)
10. Create PR on GitHub?
11. Review and testing byteammates
12. PR approved?
13. Merge branch into mainbranch
14. Clean up: Delete mergedbranches
15. End

