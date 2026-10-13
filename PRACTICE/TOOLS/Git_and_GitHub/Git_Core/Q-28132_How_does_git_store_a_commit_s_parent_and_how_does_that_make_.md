---
type: "practice"
difficulty: "Intermediate"
---

# How does git store a commit's parent, and how does that make history a directed acyclic graph?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reverting Merges](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverting_Merges.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this answer, we will delve into the inner workings of Git to understand how it stores a commit's parent reference, which forms the directed acyclic graph (DAG) nature of Git history.

TL;DR:

Git stores a commit’s parent via its object ID in the commit metadata. This relationship creates a DAG, allowing Git to reconstruct and traverse the complete history of your project.

#### Triaging the Question

The question is asking about the internal mechanism that Git uses to link commits together, forming a directed acyclic graph (DAG). The interviewer likely wants you to explain how Git stores these parent-child relationships and why they result in a DAG structure.

Step-by-Step Solution from First Principles

Commit Structure: Each commit in Git is represented as an object that contains several pieces of information, including the author name, date, message, tree (representing file changes), and a list of parent commits.

Parent References: When you create a new commit, its metadata includes references to one or more parent commits. For example, in a regular commit, it might reference exactly one parent; in a merge commit, it would reference two parents.

Commit ID and Parent Linkage: The unique identifier (SHA-1 hash) for each commit is generated based on the content of its tree object and its parent commits. This means that if you change any file or the commit message, the SHA-1 hash will differ. Importantly, these hashes are stored in the commit metadata.

Building the DAG:

When a new commit is created, Git writes this commit to the repository.
The reference (SHA) of the parent commit is stored in the newly created commit’s metadata.
This process forms a chain or branches when commits have multiple parents (merge commits).

Traversal and DAG Properties: Since each commit points to zero or more previous commits, this setup ensures that there are no cycles in the graph. This is what makes it "directed" and "acyclic."

#### Closing Summary

By storing references within commits, Git creates a directed acyclic graph of your project history. Each commit links back to its parent(s), allowing for efficient traversal and reconstruction of the full history.

#### Common Mistakes/Follow-Ups
Common Mistake: Assuming that each commit is just a standalone object without reference to others.

#### Follow-Up
The interviewer might ask about how Git handles rebase or cherry-pick operations, which can impact parent references. This would prompt further discussion on how Git manages these operations while maintaining the DAG integrity.
Code Blocks
Commit 1
Commit 2
Commit 3
Commit 4

This Mermaid diagram illustrates a simple example of how commits can link to each other, forming a DAG.

