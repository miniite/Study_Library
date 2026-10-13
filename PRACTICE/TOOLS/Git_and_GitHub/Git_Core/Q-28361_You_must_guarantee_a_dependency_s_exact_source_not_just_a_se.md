---
type: "practice"
difficulty: "Deep"
---

# You must guarantee a dependency's exact source, not just a semver, pinned to a Git commit for a supply-chain-sensitive service. Compare package-manager git refs, submodules, and vendoring for reproducibility and security.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Protected Branches and Required Checks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Protected_Branches_and_Required_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a dependency's exact source is locked down in a supply-chain-sensitive service, we need to explore three main strategies: package-manager git refs, submodules, and vendoring. Each has its own trade-offs and use cases when it comes to reproducibility and security.

First, let’s summarize the key concepts:

Package-Manager Git Refs: Using a specific Git reference (like sha or branch) in your package manager configuration.
Submodules: Including a separate repository as a subdirectory of another project.
Vendoring: Downloading and embedding all dependencies directly into the source code.

Now, let's delve into each approach:

#### Triage

The question is essentially asking how to ensure that a specific version of a dependency is locked down in your project, especially when security is critical. The interviewer will likely be probing your understanding of different Git-based dependency management strategies and their implications on reproducibility and security.

Package-Manager Git Refs

#### Assumptions
We are using a package manager like npm (Node.js), pip (Python), or similar.
The project has dependencies that need to be locked down securely.

#### First Principles Solution

Identify the Dependency: Determine which dependency needs exact version locking.

Fetch Git Commit Hash: Obtain the specific commit hash from the repository’s history. This can usually be found in release notes or through a CI/CD pipeline.

Update Configuration File:

JSON
// Example for npm
{
  "dependencies": {
    "example-dependency": "git+https://github.com/user/repo.git#commit-hash"
  }
}

Commit Changes: Ensure the package-lock.json (or equivalent) file is updated and committed to version control.

Submodules

#### Assumptions
The project needs a specific commit from another repository.
This repository might be used for multiple projects or is part of a monorepo structure.

#### First Principles Solution

Add Submodule: Add the other repository as a subdirectory within your main project.

SH
git submodule add https://github.com/user/repo.git path/to/subdir

Commit Changes:

SH
git commit -m "Added submodule for example-dependency"

Update Submodule: Ensure the submodule is updated to a specific commit.

SH
cd path/to/subdir
git checkout commit-hash
cd ..
git add path/to/subdir
git commit -m "Updated submodule to specific version"
Vendoring

#### Assumptions
You want to embed the dependency directly into your project’s codebase.
This can improve local execution speed but sacrifices reproducibility.

#### First Principles Solution

Download Source Code: Clone the repository and check out the desired commit.

Copy Files: Copy the necessary files into your project directory.

Update Dependencies: Manually update any configuration to point to these local files.

SH
git clone https://github.com/user/repo.git path/to/local-codebase
cd path/to/local-codebase
git checkout commit-hash

# Copy necessary files to your project
cp -r path/to/local-codebase/* /path/to/your/project

Commit Changes:

SH
git add /path/to/your/project/*
git commit -m "Vendor example-dependency at specific version"

#### Closing Summary
and Common Mistakes

Using package-manager git refs is often the simplest and most flexible approach for securing exact versions of dependencies without sacrificing reproducibility. However, it relies on maintaining an accurate history of commits in your own repository.

Submodules provide a good balance between flexibility and security but can introduce complexity in managing multiple repositories. Vendoring offers the best performance locally but lacks the convenience of version control for dependencies.

#### Common Mistakes
Forgetting to update the package-lock.json or equivalent file, leading to inconsistent behavior across environments.
Not keeping submodules and vendored code up-to-date with security patches.
Overlooking the impact on reproducibility when using vendoring, as it ties your project tightly to specific versions.

**Code:**

```json
{
  "dependencies": {
    "example-dependency": "git+https://github.com/user/repo.git#commit-hash"
  }
}
```

