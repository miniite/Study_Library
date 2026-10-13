---
type: "practice"
difficulty: "Deep"
---

# You want new engineers to check out only the ingestion service subtree of a large data monorepo, but their dbt build references shared macros outside the cone. Explain cone-mode sparse-checkout, how it composes with partial clone, and how to handle the out-of-cone dependency.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sparse Checkout and Monorepos](<../../../../TOOLS/Git_and_GitHub/Git_Core/Sparse_Checkout_and_Monorepos.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure new engineers can check out only the necessary parts of a large data monorepo while still having access to shared macros, we need to use Git's cone-mode sparse-checkout and partial clone features effectively. Cone-mode sparse-checkout allows us to define a subtree that will be checked out, whereas partial clone helps in reducing the size of the repository by only downloading the specified parts initially.

Cone-mode sparse-checkout is particularly useful when you want to work on a specific part of your project while keeping other parts of the repository untouched. This is especially valuable for maintaining a clean and focused working environment, which can reduce build times and improve developer productivity.

First, let's understand the core concepts:

Sparse Checkout: Allows you to specify the exact files or directories that should be cloned from a Git repository.
Partial Clone: Enables Git to clone only specific parts of the repository, reducing the initial download time by fetching only the necessary content initially.

Now, let’s see how we can apply these concepts:

Define the Ingestion Service Subtree:

The new engineers should have a clear understanding that they need to focus on the ingestion service subtree but still access shared macros.
We'll use git sparse-checkout to configure Git to only clone the necessary files.

Set Up Sparse Checkout Configuration:

BASH
git init my-repo
cd my-repo
git remote add origin https://github.com/your-organization/data-monorepo.git
git config filter.cone-core.clean 'git-cone --incone-only'
git config filter.cone-core.smudge 'cat'

Define the Cone:

To define which parts of the repository should be checked out, we can use Git's sparse-checkout feature.
BASH
git sparse-checkout init --cone
echo "path/to/ingestion-service/*" > .git/info/sparse-checkout

Handle Out-of-Cone Dependencies:

Shared macros are often stored in a central location, and engineers might need to reference these macros outside the cone.
To resolve this, we can use git filter-branch or create symlinks within the cone that point to the shared macro files.

Partial Clone for Initial Setup:

Initially, only clone the ingestion service subtree using partial clone:
BASH
git clone --filter=blob:none --no-checkout https://github.com/your-organization/data-monorepo.git my-repo
cd my-repo

Finalize Sparse Checkout:

After cloning, initialize and set up sparse checkout as described above.

Sync with Central Repository:

BASH
git config core.sparsecheckout true
echo "path/to/ingestion-service/*" > .git/info/sparse-checkout
git read-tree --empty -mu

Build and Validate Setup:

Build the dbt project to ensure that dependencies are correctly resolved within the cone.
BASH
dbt build

By following these steps, new engineers can focus on specific parts of the repository while still having access to shared resources like macros without bloating their local repository.

#### Common Mistakes
and Follow-Ups
Incorrect Cone Definition: Failing to correctly define the cone can result in missing dependencies or unnecessary files.
Overly Aggressive Partial Clone: Using partial clone too aggressively might lead to performance issues if not managed properly.
Ignoring Symlinks and Aliases: Not setting up symlinks for out-of-cone dependencies can cause build failures.

Understanding these concepts thoroughly will help in managing large repositories efficiently while maintaining a clean and focused development environment.

