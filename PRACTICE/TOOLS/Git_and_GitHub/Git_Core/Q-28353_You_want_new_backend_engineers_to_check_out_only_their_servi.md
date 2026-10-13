---
type: "practice"
difficulty: "Deep"
---

# You want new backend engineers to check out only their service subtree in a large monorepo. Explain cone-mode sparse-checkout, how it composes with partial clone, and the gotchas when a build references files outside the cone.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sparse Checkout and Monorepos](<../../../../TOOLS/Git_and_GitHub/Git_Core/Sparse_Checkout_and_Monorepos.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address your requirement for new backend engineers to check out only their service subtree in a large monorepo, we'll explore cone-mode sparse-checkout and its interaction with partial clone, while highlighting potential pitfalls when builds reference files outside the cone.

First, let's break down the key concepts:

TL;DR
Cone-mode Sparse-Checkout: Allows you to fetch only a specific subtree of a monorepo.
Partial Clone: Fetches only necessary parts of a large repository.
Common Gotchas: Builds referencing files outside the cone can cause issues, requiring careful configuration and awareness.

#### Triage
Understanding the Question

The question is asking how we can configure Git to limit the checkout scope in a monorepo environment where different services are located as separate subtrees. The goal is to enable new engineers to only clone or check out the necessary parts of the repository without pulling down the entire monorepo, which could be time-consuming and resource-intensive.

Cone-mode Sparse-Checkout

Cone-mode sparse-checkout allows you to specify a subtree of a repository that should be fetched during a git clone. This is particularly useful in large monorepos where different services are organized into separate directories. By configuring the cone correctly, we can limit the amount of data pulled down.

Step-by-Step Solution

Initial Setup:

Ensure your Git version supports sparse-checkout (Git 2.23+).
Use git clone --filter=blob:none <repo-url> to start a partial clone.

Configuring Sparse-Checkout:

SH
git config core.sparsecheckout true
echo "path/to/subtree" > .git/info/sparse-checkout

Using git sparse-checkout:

git sparse-checkout init --cone: Initializes the cone mode.
echo "path/to/subtree" >> .git/info/sparse-checkout: Add paths to be checked out.

Pulling Changes:

SH
git pull origin main
Partial Clone

A partial clone fetches only the necessary parts of a large repository, which is particularly useful for monorepos with multiple services. This can significantly reduce initial clone times and disk usage.

Step-by-Step Solution

Start a Partial Clone:

SH
git clone --filter=blob:none <repo-url>

Initialize Sparse Checkout Cone:

Follow the steps outlined for cone-mode sparse-checkout above.
Gotchas and Common Pitfalls

When builds reference files outside the checked-out subtree, you may encounter issues. Here are some key gotchas to watch out for:

Build Dependencies: If a build process requires files from other subtrees, it might fail because those files are not present in the checkout.
Dependency Management: Ensure that dependency management tools (like npm, yarn, etc.) are configured to use local copies of dependencies within the subtree.
CI/CD Pipelines: CI/CD pipelines often need to be aware of the sparse-checkout setup and may require adjustments.

#### Closing Summary

Cone-mode sparse-checkout combined with partial clone is a powerful way to manage large monorepos, especially when different services are organized into separate subtrees. However, it requires careful configuration and awareness of potential pitfalls, particularly around build dependencies and CI/CD pipelines.

#### Common Mistakes/Follow-Ups
Forgetting to initialize cone-mode: Always run git sparse-checkout init --cone after setting up sparse-checkout.
Not updating .git/info/sparse-checkout: Ensure that the paths are correctly added or updated as needed.
Ignoring build dependencies: Verify that your build system and CI/CD pipelines handle files outside the checked-out subtree.

**Comparison:**

```
Subtree	Description
service1	Contains the backend logic for a specific microservice.
service2	Contains frontend assets and UI components.
common-utils	Shared utilities used across multiple services.
```

**Code:**

```
SH
Copy
# Initial setup for partial clone and sparse-checkout
git clone --filter=blob:none <repo-url>
git config core.sparsecheckout true
echo "path/to/subtree" > .git/info/sparse-checkout

# Initializing cone mode
git sparse-checkout init --cone

# Adding paths to the sparse-checkout list
echo "path/to/subtree" >> .git/info/sparse-checkout

# Pulling changes with sparse-checkout enabled
git pull origin main
```

**Example input / output:**

```text
# Example of setting up sparse-checkout and partial clone
$ git clone --filter=blob:none https://github.com/example/large-monorepo.git my-monorepo
Cloning into 'my-monorepo'...
$ cd my-monorepo/
$ git config core.sparsecheckout true
$ echo "service1" > .git/info/sparse-checkout
$ git sparse-checkout init --cone
$ git pull origin main
remote: Enumerating objects: 234, done.
Receiving objects: 100% (234/234), 79.56 KiB | 487.00 KiB/s, done.
Resolving deltas: 100% (84/84), done.
Checking out files: 100% (48/48), done.
```

**Data:**

```json
{
  "repo-url": "https://github.com/example/large-monorepo.git",
  "subtree": "service1"
}
```

**Flow:**

1. Initial Setup
2. Partial Clone
3. Sparse Checkout Init --cone
4. Add Paths to Sparse-Checkout
5. Pull Changes

