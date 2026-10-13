---
type: "practice"
difficulty: "Intermediate"
---

# Why is it a bad idea to commit a large local Parquet output or a DuckDB database file, and how would DVC or Git LFS help if you truly must track a sample?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Git LFS](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_LFS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, committing large files like local Parquet outputs or a DuckDB database directly into Git can lead to significant issues due to Git's design and workflow constraints. Git is optimized for version control of small text files; handling large binary files such as Parquet outputs or entire databases can result in bloated repositories, slow performance, and difficulties in collaborating with others.

To understand why these approaches are problematic, let’s break down the key points:

Repository Size: Large files like Parquet or full database dumps significantly increase the repository size. This makes cloning and updating the repository slower and more resource-intensive.
Performance Issues: Git is not designed to handle large binary files efficiently. Operations such as git commit, git pull, and git push can become extremely slow when dealing with large files, leading to frustrating developer experiences.
Collaboration Challenges: Collaborating on a repository containing large files becomes cumbersome. Developers often find it difficult to work in a branch or merge changes without running into performance bottlenecks.

#### Triage

The core of the question revolves around understanding why certain practices are not ideal and exploring alternatives like DVC (Data Version Control) and Git LFS (Large File Storage). We need to ensure that we have a clear grasp of these tools' capabilities and limitations before proposing solutions.

#### First-Principles Solution
Committing Large Local Parquet Outputs

When working with large datasets, it is generally recommended not to commit the actual data files into the Git repository. Instead, you can store metadata or configuration files that point to where the data resides. Here’s a step-by-step approach:

Data Storage: Store your Parquet files in an external storage service like Amazon S3, Google Cloud Storage, or local disk with proper permissions.
Metadata Management: Create a JSON file containing paths and relevant metadata about the data stored externally.
Git Committing: Only commit the metadata file into Git.
Using DVC

DVC (Data Version Control) is specifically designed to handle large datasets while maintaining the integrity of your Git workflow. Here’s how it works:

DVC Configuration: Define a dvc.yaml file that describes the data flow and dependencies.
Tracking Data: Use dvc add <path_to_data> to track the external data files without committing them directly into Git.
Git Committing: Commit the DVC configuration and metadata files to your repository.

#### Example
Workflow

Here’s a simplified example using Parquet files stored in S3:

S3 Storage:

PLAINTEXT
s3://my-bucket/data/parquet_data.parquet

DVC Configuration (dvc.yaml):

YAML
repo: .
stages:
  - name: parquet_dataset
    cmd: cp data/parquet_data.parquet s3://my-bucket/data/
    deps:
      - data/parquet_data.parquet
    outs:
      - s3://my-bucket/data/parquet_data.parquet

Committing to Git:

BASH
git add dvc.yaml .dvc/.gitignore
git commit -m "Added Parquet dataset tracking"
Using Git LFS

Git Large File Storage (LFS) is another solution for handling large binary files in Git repositories. Here’s a brief overview:

Install Git LFS:

BASH
git lfs install

Configure Git LFS: Create a .gitattributes file to specify which files should be managed by Git LFS.

PLAINTEXT
data/parquet_data.parquet filter=lfs diff=lfs merge=lfs -text

Commit Changes:

BASH
git add .gitattributes
git commit -m "Configured Git LFS for Parquet dataset"
git lfs track "*.parquet"
git add data/parquet_data.parquet
git commit -m "Added large Parquet file to Git LFS"

#### Closing Summary
and Common Mistakes/Follow-Ups

By using DVC or Git LFS, you can effectively manage large datasets without cluttering your Git repository. This approach ensures that your workflow remains efficient and collaborative.

#### Common mistakes
include:

Overlooking the performance implications of committing large files.
Failing to properly configure metadata tracking with DVC.
Misusing Git LFS by not specifying which files should be managed.

In follow-ups, it would be useful to discuss scenarios where you might prefer one tool over the other based on specific project requirements and team practices.

