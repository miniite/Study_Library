---
type: "practice"
difficulty: "Deep"
---

# You want new engineers to clone only the part of the monorepo they work on. Explain sparse-checkout (cone mode) setup, how it interacts with partial clone, and the gotchas when files outside the cone are referenced.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sparse Checkout and Monorepos](<../../../../TOOLS/Git_and_GitHub/Git_Core/Sparse_Checkout_and_Monorepos.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Sparse-checkout is a powerful feature in Git that allows you to work on specific parts of a monorepo without downloading the entire repository. This setup is particularly useful for new engineers who might only be working on one or two projects within a large monorepo, as it optimizes storage and bandwidth usage by cloning only relevant files.

#### Triaging the Question

The question is primarily probing your understanding of Git sparse-checkout and how it interacts with partial clones. Sparse-checkout can significantly reduce the repository size for individual contributors, making it easier to manage large monorepos. The interviewer will likely be interested in how you set up and maintain this environment, as well as any potential pitfalls.

Setting Up Sparse-Checkout (Cone Mode)

Sparse-checkout uses "cone mode" to define a specific directory structure that will be cloned from the remote repository. Here's a step-by-step guide on setting it up:

Clone the Repository: Start by cloning the monorepo using git clone with the --single-branch option.
BASH
git clone --single-branch https://github.com/example/multirepo.git
Initialize Sparse Checkout:
Navigate to your local repository directory.
Use git config core.sparsecheckout true to enable sparse checkout mode.
BASH
cd multirepo
git config core.sparsecheckout true
Define the Cone: Create a .git/info/sparse-checkout file and list the paths of the directories you want to include in your clone.
For example, if new engineers only need client/ and common/lib/, create a sparse checkout definition:
PLAINTEXT
client/
common/lib/
Update the Index: After setting up the cone file, update the index to reflect the changes.
BASH
git read-tree -mu HEAD
Interactions with Partial Clone

A partial clone is another optimization that can be used in conjunction with sparse-checkout. It involves only downloading the commit history relevant to your branch and the files within your cone.

Initial Setup: When you first git clone --single-branch, it starts as a full clone, but using --filter=blob:none helps reduce initial download size.
BASH
git clone --single-branch --filter=blob:none https://github.com/example/multirepo.git
Partial Clone: Git will only fetch the necessary commits and blobs when you perform operations like git pull. This means that other branches and directories are not downloaded until needed.
Gotchas and Common Pitfalls
File References Outside the Cone:
If your work involves referencing files outside of the cone (e.g., importing modules from another project), Git will complain when you try to checkout or stage those files.
BASH
git: 'client/common/lib/somefile.js' is not in the tree
Branch Merges and Collaborative Work:
When working on branches that reference files outside your cone, you may need to manually pull updates from the remote repository.
Performance Overhead: While sparse-checkout saves space, operations like git log or git blame might be slower as they still traverse through all available history in the repository.

#### Closing Summary

Sparse-checkout and partial clone together provide a powerful way to manage large monorepos by only downloading relevant files. However, it’s essential to understand its limitations, especially when dealing with file references outside the cone. Proper setup and management can save significant resources and improve developer productivity.

#### Common Mistakes/Follow-Ups
Failing to properly define the .git/info/sparse-checkout file.
Not using --filter=blob:none during initial cloning for partial clones.
Ignoring file references outside the cone, leading to unexpected errors or missing files.
Misunderstanding the performance implications of sparse checkout on certain operations.

