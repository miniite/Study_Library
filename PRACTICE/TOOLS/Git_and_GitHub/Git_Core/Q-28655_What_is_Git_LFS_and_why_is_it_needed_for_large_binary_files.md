---
type: "practice"
difficulty: "Deep"
---

# What is Git LFS and why is it needed for large binary files?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Git LFS](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_LFS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Git Large File Storage (LFS) is an extension to Git that addresses the issue of storing large binary files, such as videos, images, or executables, in a version control system like GitHub. The need for LFS arises because traditional Git can only efficiently handle small text-based files and struggles with large files due to their size and impact on repository performance.

#### Triage

The question is essentially asking you to explain the purpose of Git Large File Storage (LFS) and why it's crucial when dealing with large binary files. The interviewer might be probing your understanding of Git's limitations, version control practices, and the trade-offs involved in using LFS versus other storage solutions.

#### First-Principles Solution

Git is designed to manage small text-based changes efficiently by storing only differences between versions ( deltas ). However, this approach fails when dealing with large binary files because it requires keeping a full copy of each version. This can lead to several issues:

Repository Size: Large files increase the size of the repository significantly.
Performance Overhead: Every time you clone or fetch from a remote, all large files are transferred unnecessarily, leading to slower operations and increased network traffic.
Storage Costs: Storing larger file sizes on the same hardware can be more expensive.

Git LFS addresses these issues by storing only references (hashes) of large binary files in Git repositories while keeping the actual file content outside the repository directory. This way, you get the benefits of version control—like branching and merging—with minimal impact on your repository size and performance.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Git LFS is essential for projects that involve large binary files because it keeps your Git repositories lean and fast while retaining all the features of a robust version control system. Key takeaways include understanding how LFS works, its benefits in terms of repository size and performance, and recognizing scenarios where traditional Git might struggle.

Some common mistakes to avoid:

Failing to understand why traditional Git is limited when dealing with large files.
Not realizing that Git LFS allows you to keep track of changes while only storing the latest version of a file on your local machine.
Overlooking how Git LFS can integrate seamlessly into existing workflows without significant disruption.
Depth + Example

To further illustrate, let's consider an example. Suppose you are working on a project with several large video files (e.g., 1 GB each). Without Git LFS, cloning the repository would require downloading all these videos, even if they have not changed between commits. With Git LFS, only a small reference to the file is stored in the commit history, reducing both the initial download size and subsequent clone times.

Self-Check

This answer covers the core concepts of Git LFS, its necessity, and practical benefits effectively.

**Comparison:**

```
Initialize LFS
Add Large File
Stage File
Commit Changes
Push to Remote
Clone Repository
Fast Clone Time
```

**Code:**

```bash
# Example command to add a large file to Git LFS
git lfs track "path/to/largefile.*"

# Example command to commit changes using Git LFS
git add path/to/largefile.jpg
git commit -m "Add new image"
```

**Example input / output:**

| Command | Input | Output |
|---------|-------|--------|
| `git lfs track "path/to/largefile.jpg"` | None | Tracks large file for LFS |
| `git add path/to/largefile.jpg` | Large file added to repository | Stages file with reference only |
| `git commit -m "Add new image"` | Commit message provided | Commits changes, storing small references in Git history |

**Data:**

```json
{
  "exampleRepository": {
    "sizeWithoutLFS": 1024 * 1024 * 1024, // 1 GB in bytes
    "sizeWithLFS": (1024 * 1024 + 3) / 1024 // Small reference in KB
  }
}
```

**Flow:**

1. No
2. Yes
3. Initial Git Repo
4. Large File Cloned
5. Repository Size Increase
6. LFS?
7. Full File Download
8. Reference Only
9. Fast Clone Time

