---
type: "practice"
difficulty: "Intermediate"
---

# Your git push of a new DAG change is rejected with a non-fast-forward error because a teammate pushed a model fix first. How do you fix it without losing either change?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Rejected Pushes](<../../../../TOOLS/Git_and_GitHub/Git_Core/Rejected_Pushes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a non-fast-forward error during a git push, it indicates that your local branch is behind the remote one and there are unmerged changes. In this scenario, you need to ensure both your change and your teammate's fix are merged correctly without losing any modifications. Here’s how you can resolve this issue step by step:

Understand the Situation: You've made a change to a DAG (Directed Acyclic Graph) in Airflow that was not pushed yet, but another team member has already pushed a model fix. Your local version is behind the remote branch.

Fetch and Merge Remote Changes:

Fetch all recent changes from the remote repository: git fetch origin.
Ensure your current working directory is clean: git checkout <branch_name>, where <branch_name> is the name of your local branch.
Pull the latest changes to bring them into your local branch: git pull origin <branch_name>. This command will automatically merge any changes from the remote repository, potentially resolving conflicts if they exist.

Rebase Your Changes:

If pulling and merging directly does not work (e.g., due to unmerged changes), you can rebase your local branch onto the updated base: git rebase origin/<branch_name>. This command will replay your commits on top of the latest version from the remote branch, potentially resolving conflicts during this process.

Resolve Conflicts:

If there are merge or rebase conflicts, you'll need to manually resolve them. Git will guide you through the process by marking conflicting files.
After resolving conflicts, add the resolved files and continue: git add <file> followed by git commit --amend.
Once all conflicts are resolved and commits are updated, force push your branch: git push origin <branch_name> --force. Note that this will override the remote history with your local changes.

Verify Changes:

After pushing, verify that both your DAG change and the model fix have been merged correctly by checking the repository on GitHub or through a pull request review process.
Ensure your DAG runs as expected in the Airflow UI to confirm no issues were introduced during the merge.

Communicate with Teammates:

Inform your team about any force push, especially if it impacts ongoing work. This helps maintain communication and prevents future conflicts.

By following these steps, you can ensure both changes are integrated without losing either one.

