---
type: "practice"
difficulty: "Deep"
---

# Developer clones of your 8 GB AIOps monorepo (dashboards, rules, runbooks, telemetry fixtures) take 15 minutes and thrash CI. Compare partial clone with --filter=blob:none, shallow clone, and sparse-checkout for this, including which operations fetch lazily and the offline failure modes during an incident.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sparse Checkout and Monorepos](<../../../../TOOLS/Git_and_GitHub/Git_Core/Sparse_Checkout_and_Monorepos.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When dealing with large monorepos like AIOps in a Git environment, cloning can be a bottleneck due to their size. To optimize this process and reduce CI thrashing, we can explore three strategies: partial clone using --filter=blob:none, shallow clone, and sparse-checkout. These methods allow for different levels of data fetching lazily, which can significantly speed up the initial clone operation while still providing necessary functionality during development or CI builds.

Triaging

The core issue here is reducing the time it takes to clone a large monorepo (8 GB) from GitHub, specifically in scenarios where developers and continuous integration systems are frequently making clones. The question asks us to compare three different strategies for improving this process:

Partial Clone using --filter=blob:none: This method filters out certain files during the initial clone.
Shallow Clone: Clones only a shallow history of commits, which is useful for read-only operations or when you need recent changes but not all past revisions.
Sparse-Checkout: Allows developers to check out specific files or directories while skipping others.

#### First Principles Solution
1. Partial Clone with --filter=blob:none

A partial clone using --filter=blob:none allows the initial clone operation to be much faster by reducing the amount of data transferred. This is particularly useful when you are primarily working on certain files or directories within a large monorepo.

How it Works: During the initial clone, only the necessary metadata and references for specific paths are fetched. The actual file content is not downloaded until it is accessed.
Operations Fetched Lazily: Any file that is touched during development or CI builds will be lazily fetched from the remote repository when needed.

#### Example
Command:

SH
git clone --filter=blob:none https://github.com/your-repo/aio-mono-repo.git
2. Shallow Clone

A shallow clone limits the history of commits that are downloaded, making it much faster for initial cloning but limiting access to older revisions.

How it Works: Only a limited number of recent commits (typically the last N commits) are fetched during the initial clone.
Operations Fetched Lazily: Older commits can be fetched later using git fetch or git pull, but this might introduce latency in CI builds if they need older data.

#### Example
Command:

SH
git clone --depth=50 https://github.com/your-repo/aio-mono-repo.git
3. Sparse-Checkout

Sparse-checkout allows you to check out specific files or directories from a monorepo, reducing the initial load and making it easier to work with large repositories.

How it Works: You specify which paths to include in your local working copy during the clone process.
Operations Fetched Lazily: Only the specified paths are cloned initially. Other files can be fetched later using git update-index or git read-tree.

#### Example
Commands:

SH
git init
git remote add origin https://github.com/your-repo/aio-mono-repo.git
git config core.sparsecheckout true
echo "dashboards/*" >> .git/info/sparse-checkout
echo "rules/*" >> .git/info/sparse-checkout
git pull --depth=1 origin master

#### Closing Summary

Each of these strategies has its own use cases and trade-offs. Partial clone with --filter=blob:none is great for reducing initial clone time while allowing access to necessary files as they are used. Shallow clone can be useful when you only need recent history, but it limits your ability to work on older revisions. Sparse-checkout is ideal for working with specific parts of a large monorepo, making development and CI builds faster.

#### Common Mistakes/Follow-Ups

Mistake: Assuming all three methods provide the same level of access or functionality.

Correctness: Each method has different implications on what data is available offline versus online. Developers must understand the limitations before choosing a strategy.

#### Follow-Up
Discussing how to handle situations where developers need older data after using shallow clone.

#### Solution
Use git fetch or git pull to retrieve older commits as needed, which may introduce latency.

Mistake: Failing to consider the overhead of lazy fetching.

Correctness: While lazy fetching can speed up initial cloning, it might slow down development if files are frequently accessed. Developers should balance clone time with access speed.
Triaging and First Principles Solution

By understanding how each strategy works, you can choose the best method based on your specific needs. Partial clones and sparse-checkout are particularly useful in CI environments where frequent builds need to be fast.

**Code:**

```
SH
Copy
# Example commands for partial clone with --filter=blob:none
git clone --filter=blob:none https://github.com/your-repo/aio-mono-repo.git

# Example command for shallow clone
git clone --depth=50 https://github.com/your-repo/aio-mono-repo.git

# Example commands for sparse-checkout
git init
git remote add origin https://github.com/your-repo/aio-mono-repo.git
git config core.sparsecheckout true
echo "dashboards/*" >> .git/info/sparse-checkout
echo "rules/*" >> .git/info/sparse-checkout
git pull --depth=1 origin master
```

**Example input / output:**

```

No specific input-output examples are needed for these methods, but here’s a general dry-run:

SH
Copy
# Partial clone with lazy fetching
$ git clone --filter=blob:none https://github.com/your-repo/aio-mono-repo.git

# Shallow clone with manual fetch of older history
$ git clone --depth=50 https://github.com/your-repo/aio-mono-repo.git
$ git pull origin <older-branch>

# Sparse-checkout for specific directories
$ git init
$ git remote add origin https://github.com/your-repo/aio-mono-repo.git
$ git config core.sparsecheckout true
$ echo "dashboards/*" >> .git/info/sparse-checkout
$ echo "rules/*" >> .git/info/sparse-checkout
$ git pull --depth=1 origin master
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Partial Clone with'--filter=blob:none'
8. File Accessed?
9. Lazily fetch from remote
10. Initial clone complete
11. Shallow Clone
12. Older Commit Needed?
13. Pull older commit with `gitpull`
14. Initial clone complete, shallowhistory only
15. Sparse-Checkout
16. Specific Path Accessed?
17. Lazily fetch specific path fromremote
18. Initial sparse checkoutcomplete

