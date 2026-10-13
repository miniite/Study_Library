---
type: "practice"
difficulty: "Intermediate"
---

# A designer keeps committing large PNG and Figma exports that bloat the frontend repo. How would you introduce Git LFS mid-project, migrate the existing binaries, and what breaks for people who don't have LFS installed?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Git LFS](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_LFS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of large binary files like PNGs and Figma exports bloating the frontend repository, we can introduce Git Large File Storage (LFS) to manage these files efficiently. This involves several steps including migration from existing local binaries, ensuring smooth transition for team members who may not have LFS installed initially, and maintaining a balance between file size and performance.

#### Triage
Understanding the Problem

The core of the issue is managing large binary files in Git repositories, which can lead to slow cloning and fetching times. Git LFS allows us to replace large files with pointers within Git, while storing the actual data on a remote server or local storage.

Step-by-Step Solution from First Principles

Understand Git LFS Basics:

Git LFS tracks large binary files by replacing them with placeholders in the commit history.
These placeholders point to the actual file content stored on an external service (e.g., a custom server or GitHub's Git LFS).

Install Git LFS Globally:

First, ensure that all team members have Git LFS installed globally by running:
BASH
git lfs install

Configure Git LFS for the Repository:

Initialize Git LFS in your repository with a specific file pattern or content type (e.g., images, Figma files). This can be done by creating a .gitattributes file:
PLAINTEXT
# .gitattributes
*.png filter=lfs diff=lfs merge=lfs -text
*.figma filter=lfs diff=lfs merge=lfs -text

Migrate Existing Binaries:

Use the git lfs migrate command to transfer existing large files into Git LFS:
BASH
git lfs migrate import --include='**/*.png' --include='**/*.figma'
This process can take a while, as it needs to download and re-stage the files.

Push Changes with LFS:

Once migration is complete, push your changes to the remote repository:
BASH
git add .gitattributes
git commit -m "Add Git LFS tracking for large files"
git push origin main

Communicate Changes to Team Members:

Inform team members about the new setup and instruct them on how to install Git LFS:
BASH
git lfs install
Provide a brief summary of the changes in a pull request or meeting, explaining why this change is necessary.

Handle Non-LFS Users Gracefully:

For users who do not have LFS installed, they can still clone and work with the repository but will not be able to check out large files locally.
They should run:
BASH
git lfs install
Alternatively, you can use Git's --depth or shallow clone options to make it easier for them:
BASH
git clone --depth 1 https://github.com/yourrepo/frontend.git

#### Closing Summary
and Common Mistakes/Follow-Ups

Summary: Introducing Git LFS is a robust solution to manage large binary files efficiently. By following these steps, we can minimize repository bloat and improve collaboration among team members.

#### Common Mistakes

Failing to inform all team members about the new setup.
Not using git lfs install when needed, leading to errors in checkout.
Overlooking performance implications for non-LFS users or repositories where LFS introduces significant overhead.

#### Follow-Ups

Implementing a system for automatic LFS migration on pull requests or branches.
Exploring alternative file management strategies like minification or compression if LFS is not suitable for certain file types.

**Code:**

```bash
# Install Git LFS globally
git lfs install

# Initialize LFS in repository
echo "*.png filter=lfs diff=lfs merge=lfs -text" >> .gitattributes
echo "*.figma filter=lfs diff=lfs merge=lfs -text" >> .gitattributes

# Migrate existing large files
git lfs migrate import --include='**/*.png' --include='**/*.figma'

# Push changes with LFS
git add .gitattributes
git commit -m "Add Git LFS tracking for large files"
git push origin main
```

**Flow:**

1. Initialize Git LFS
2. Create .gitattributes
3. Migrate large files with `git lfsmigrate`
4. Push changes to remoterepository
5. Inform team members aboutnew setup
6. Install Git LFS for non-LFSusers

