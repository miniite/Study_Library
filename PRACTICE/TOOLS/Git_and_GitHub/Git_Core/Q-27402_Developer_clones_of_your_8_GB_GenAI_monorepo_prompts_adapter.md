---
type: "practice"
difficulty: "Deep"
---

# Developer clones of your 8 GB GenAI monorepo (prompts, adapters, eval data) take 15 minutes. Compare partial clone with --filter=blob:none, shallow clone, and sparse-checkout, including what fetches lazily.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sparse Checkout and Monorepos](<../../../../TOOLS/Git_and_GitHub/Git_Core/Sparse_Checkout_and_Monorepos.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with large repositories like an 8 GB GenAI monorepo that takes 15 minutes to clone, optimizing the cloning process is crucial. This scenario involves comparing three strategies: --filter=blob:none, shallow clones, and sparse-checkout. Each approach has its own trade-offs in terms of disk usage, network bandwidth, and eventual checkout time.

First, let's understand what each method does:

--filter=blob:none: This flag tells Git to fetch the necessary metadata but not the actual file contents (blobs) for the specified refspecs. The blobs will be fetched lazily when you first access them.
Shallow Clones: A shallow clone is a lightweight version of the repository that only includes commits up to a certain depth, typically the latest commit or a specific tag/branch. This means it won't have all the history and can be useful for read-only operations but not suitable for making changes.
Sparse-Checkout: Sparse-checkout allows you to clone a repository with only a subset of files/folders checked out initially. The rest will be lazily fetched when you need them.

Let's break down each strategy step-by-step:

--filter=blob:none:

This method fetches metadata but not the blobs (file contents) for the specified ref.
The blobs are only fetched on-demand when accessed, which can reduce initial download time and memory usage.
Use case: Ideal for read-only access or scenarios where you're planning to make significant changes later.

Shallow Clones:

A shallow clone will contain a limited history (e.g., just the latest few commits).
This approach is faster in terms of initial download time and disk space.
Limitations: Cannot be used for operations that require full history, such as rebasing or committing new changes.

Sparse-Checkout:

Sparse-checkout allows you to specify a subset of files/folders to clone initially.
The rest are fetched lazily when needed.
Use case: Ideal for projects with large file structures where only specific parts need to be accessed frequently.

Here's a sample workflow using git commands:

SH
# For --filter=blob:none
git clone --filter=blob:none <repo-url>

# For shallow clone
git clone --depth 1 <repo-url> # Clones up to the latest commit

# For sparse-checkout
git clone <repo-url>
cd <project-root>
git config core.sparsecheckout true
echo "prompts/" > .git/info/sparse-checkout # Example path
git read-tree -mu HEAD

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Using shallow clones for operations that require full history, such as creating branches or making commits.

#### Follow-Up
Discussing how to handle situations where you need to switch between different parts of the repository after using sparse-checkout.

Overall, each method has its strengths and weaknesses. --filter=blob:none is good for initial read-only access, shallow clones are great for quick prototyping or testing with recent history, and sparse-checkout is ideal for working with large projects where only specific parts need to be accessed frequently.

