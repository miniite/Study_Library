---
type: "practice"
difficulty: "Intermediate"
---

# A designer keeps committing large model-eval screenshots and sample audio into the GenAI repo, bloating it. How would you introduce Git LFS mid-project, migrate existing binaries, and what breaks for people without LFS installed?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Git LFS](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_LFS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of large files in a Git repository like GenAI, we can introduce Git Large File Storage (LFS) to manage these big assets efficiently. This approach will ensure that only metadata is stored directly in the Git history while the actual file data is kept on a remote server. Here’s how you can migrate existing large files and set up LFS:

First, let's understand what Git LFS does: It replaces every large file in your repository with a placeholder (a blob ID) which Git tracks normally. The actual content of these files gets stored separately on the cloud provider's servers.

Now, let’s dive into the steps required to migrate and set up Git LFS:

Install Git LFS: Ensure that both the local developer environment and the remote repository server have Git LFS installed.
Initialize Git LFS in the Repo:
SH
git lfs install
Track Large Files: Identify which file types should be managed by Git LFS, typically large files like images, audio, and video.
Migrate Existing Large Files to LFS:
SH
git lfs track "*.mp3"

This command tells Git that any .mp3 file found in the repo should now use Git LFS for storage.

Commit Changes: Commit your changes with git add . && git commit -m "Add support for LFS and start tracking mp3 files". This will update the index to reflect the new tracking rules.

Push to Remote Repository:

SH
git push origin main

After pushing, existing large files will be converted into Git LFS pointers in your repository history.

Configure Git LFS on Remote Server: If you're using a private Git server like GitHub or Bitbucket, make sure it has LFS support enabled. For GitHub, this is handled automatically when the repo has LFS content.
Inform Team Members: Clearly communicate to all team members that:
They need to install Git LFS on their local machine if they haven't already.
The repository now uses LFS for large files, so git pull and git push might take longer due to the extra data transfer.

Here’s a worked example of an input-output scenario:

SH
# Step 1: Install Git LFS on developer's machine
$ git lfs install

# Step 2: Track mp3 files for LFS
$ git lfs track "*.mp3"

# Step 3: Commit the tracking rule changes
$ git add . && git commit -m "Add support for LFS and start tracking mp3 files"

# Step 4: Push the changes to the remote repository
$ git push origin main

# Step 5: Pull changes on a colleague's machine (after they install Git LFS)
$ git pull origin main

#### Common Mistakes

Not installing Git LFS before starting the migration process.
Failing to inform all team members about the new setup and how it affects their local repositories.

**Code:**

```
SH
Copy
git lfs install
git lfs track "*.mp3"
git add . && git commit -m "Add support for LFS and start tracking mp3 files"
git push origin main
```

