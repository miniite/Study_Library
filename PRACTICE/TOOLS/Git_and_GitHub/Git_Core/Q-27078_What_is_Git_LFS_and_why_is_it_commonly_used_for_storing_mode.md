---
type: "practice"
difficulty: "Intermediate"
---

# What is Git LFS and why is it commonly used for storing model weights or images in an ML repository?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Git LFS](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_LFS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Git Large File Storage (LFS) is a feature of Git that allows you to store large files, such as model weights or image datasets, outside of your main Git repository while still tracking them in a Git branch. This solution helps manage large file repositories efficiently and keeps the primary repository small and manageable.

#### Triage

The question asks about Git LFS and its common usage for storing model weights or images in an ML repository. The key points are understanding what Git LFS is, why it's necessary, and how it addresses specific challenges faced by developers using Git for machine learning projects.

#### First-Principles Solution

Understanding Git LFS: Git LFS is designed to handle large files that exceed the typical 100MB limit imposed by standard Git. When you install Git LFS, a hook is set up in your repository. This hook intercepts pushes and pulls, replacing the actual large file content with a small reference (pointer) inside the Git commit history.

Why Use Git LFS:

Space Efficiency: Storing model weights or images directly in a Git repo can quickly consume a lot of space, making it impractical for repositories that need to be shared and cloned often.
Performance: Large files stored within Git commits can slow down cloning and synchronization processes. Git LFS ensures that only the pointers are stored in your repository, speeding up these operations.
Version Control: With Git LFS, you maintain version control over large files just like with other tracked files. You can revert to previous versions of a model or dataset if needed.

#### Implementation
Steps:

Install Git LFS:
BASH
git lfs install
Configure Git LFS for Specific File Types (e.g., images):
BASH
git lfs track "*.png"
git lfs track "*.jpg"
Push and Pull Large Files: When pushing or pulling, Git LFS will handle the transfer of large files.
Checkout the Repository Locally:
BASH
git clone <repository-url>

#### Closing Summary
+ Common Mistakes/Follow-Ups

Git LFS is a valuable tool for managing large files in ML repositories by keeping your main Git repository small and fast while still tracking versions of these files. Common mistakes include not configuring Git LFS properly, leading to issues when trying to work with the large files. Always ensure you have installed and configured Git LFS correctly before committing any large files.

#### Follow-ups
might include discussing how to integrate Git LFS with CI/CD pipelines or handling scenarios where multiple developers are working on the same repository.

**Code:**

```bash
# Install Git LFS
git lfs install

# Track specific file types
git lfs track "*.png"
git lfs track "*.jpg"

# Push and pull large files
git push
git pull
```

