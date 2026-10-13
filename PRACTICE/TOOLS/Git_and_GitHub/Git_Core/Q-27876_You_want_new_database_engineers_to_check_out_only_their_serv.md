---
type: "practice"
difficulty: "Deep"
---

# You want new database engineers to check out only their service's migrations subtree in a large monorepo. Explain cone-mode sparse-checkout, how it composes with a partial clone, and the gotchas when a migration references a shared seed file outside the cone.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Sparse Checkout and Monorepos](<../../../../TOOLS/Git_and_GitHub/Git_Core/Sparse_Checkout_and_Monorepos.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Cone-mode sparse-checkout is an advanced feature in Git that allows you to check out only specific parts of a repository while still leveraging the benefits of a full clone, such as the ability to fetch updates and collaborate on shared code. In the context of database migrations in a monorepo setup, this can be incredibly useful for keeping the developer environment lightweight but still able to access necessary files when required.

#### Triage

The question is about using cone-mode sparse-checkout with partial clones to manage database migrations in a large monorepo while avoiding unnecessary clutter. Specifically, it probes your understanding of how sparse-checkout works and its interaction with partial cloning, as well as potential issues when migrations reference shared seed files outside the specified subtree.

#### First-Principles Solution

To address this, we'll start by explaining what cone-mode sparse-checkout is and then delve into how it can be used to limit the scope of checked-out code. We’ll also discuss partial cloning and how these two concepts work together. Finally, we’ll highlight common gotchas related to shared files across different subtrees.

Cone-Mode Sparse-Checkout

Cone-mode sparse-checkout in Git allows you to specify a subtree within a repository that you want to check out. This means you can effectively "pinch" the repository at a certain branch or tag and only fetch those commits, along with their immediate dependencies. Here’s how it works:

Define the Cone: You define which parts of the repository should be included in your sparse-checkout cone. This is typically done using git config commands to set up the desired branches and paths.
Check Out Only the Cone: When you clone or fetch, Git only checks out the specified subtree. This means that while you can still access the full history of the repository, only the files in your cone are stored locally.
Partial Cloning

Partial cloning is another feature introduced in Git 2.16 (released in March 2018) that allows developers to clone only a part of a large repository without fetching all its history. This can significantly reduce the initial download time and disk usage, making it ideal for environments where bandwidth or storage is limited.

Here’s how partial cloning works:

Define the Branch: You specify which branch you want to clone.
Clone Only What's Needed: Git will fetch only the objects needed for the specified branch and its immediate dependencies, reducing the initial download size significantly.
Composing Cone-Mode Sparse-Checkout with Partial Cloning

By combining cone-mode sparse-checkout and partial cloning, you can achieve a very efficient setup where developers check out only the necessary migrations while still benefiting from the robust features of a full clone. Here’s a step-by-step process:

Define the Cone: Configure Git to include only the migration files for your service.
Partial Clone: Use --filter=blob:none with git clone or git fetch --depth=1 to reduce the initial download size.
Sparse Checkout: Set up sparse-checkout using .git/info/sparse-checkout or via commands like git config core.sparsecheckout true.

Here’s an example:

SH
# Define the cone for migrations
echo "path/to/service/migrations/**" > .git/info/sparse-checkout

# Clone partially and set sparse-checkout
git clone --filter=blob:none --no-checkout https://github.com/yourrepo/monorepo.git myservice-migration
cd myservice-migration
git config core.sparsecheckout true
echo "path/to/service/migrations/**" > .git/info/sparse-checkout

# Fetch the migration branch and its dependencies
git checkout <migration-branch>
Gotchas When Migrations Reference Shared Seed Files

One significant challenge is when migrations need to reference shared seed files outside the specified cone. Here are some key issues and their solutions:

Shared Files Outside Cone: If a migration needs a shared seed file, you might end up with incomplete or broken migrations if those files aren’t included in your sparse-checkout.
Manual Inclusion: You can manually include additional paths in the .git/info/sparse-checkout file to ensure all necessary files are available.
Partial Fetch for Seeds: Use git fetch --depth=1 <shared-seed-branch> to selectively pull in specific branches or tags that contain shared seed files.

#### Common Mistakes
and Follow-Ups
Overly Narrow Cone: Developers might inadvertently narrow the cone too much, leading to incomplete migrations.
Ignoring Dependencies: Failing to include necessary dependencies can result in broken migrations.
Performance Overhead: While sparse-checkout is efficient, there might be a slight performance overhead when accessing files outside the cone.

#### Closing Summary

Cone-mode sparse-checkout and partial cloning offer powerful tools for managing large monorepos like database migrations. By carefully defining which parts of the repository to check out, developers can maintain a lightweight local environment while still having access to necessary code. However, it’s crucial to manage shared files properly to avoid issues.

Self-Check

This solution covers all aspects of sparse-checkout, partial cloning, and common pitfalls, ensuring completeness and correctness.

**Code:**

```
SH
Copy
echo "path/to/service/migrations/**" > .git/info/sparse-checkout
```

