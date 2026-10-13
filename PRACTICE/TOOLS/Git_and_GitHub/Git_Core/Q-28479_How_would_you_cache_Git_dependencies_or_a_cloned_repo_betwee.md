---
type: "practice"
difficulty: "Intermediate"
---

# How would you cache Git dependencies or a cloned repo between CI runs to speed up builds?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ CI Cache Correctness](<../../../../TOOLS/Git_and_GitHub/Git_Core/CI_Cache_Correctness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To speed up Git dependency and cloned repo caching in CI (Continuous Integration) runs, we need to understand that reducing redundant network operations is key. This can significantly enhance build times by reusing cached content from previous builds. The solution involves using Git LFS (Large File Storage), Git submodules, or leveraging GitHub Actions' built-in caching mechanisms.

#### Triaging the Question

The primary goal of the question is to optimize CI runs by minimizing network latency and reducing unnecessary data transfer. Specifically, we are looking for strategies that can cache Git dependencies and cloned repos efficiently between builds. The assumptions include:

We have a CI/CD pipeline in place.
We use GitHub as our version control system.

#### First Principles Solution

To achieve caching, we need to consider different scenarios based on the nature of the repository contents. Here’s how you can implement it:

1. Using Git LFS for Large Files
If your repo contains large files (e.g., videos, images), use Git LFS. This stores these files outside the main Git history and uses a pointer in place.
Cache Git LFS objects locally or on a shared storage system.
2. Using Submodules for Independent Repositories
If your project includes other repositories as submodules (e.g., dependencies), clone them separately and cache their contents.
Keep track of submodule versions using .gitmodules and update the cached repo accordingly.
3. Leveraging GitHub Actions Caching Mechanism
Utilize actions/cache to cache Git repos or specific directories within a repo.
Define cache keys that uniquely identify the state of your project (e.g., based on commit hash, branch name).

#### Implementation
Steps
Step 1: Setup LFS Cache
YAML
- name: Clone the repository and install dependencies
  uses: actions/checkout@v2
  with:
    fetch-depth: 0
Step 2: Configure Git Submodules
YAML
- name: Initialize submodules
  run: git submodule update --init --recursive
Step 3: Use actions/cache for Dependency Caching
YAML
- name: Cache dependencies
  id: cache_dependencies
  uses: actions/cache@v2
  with:
    path: vendor
    key: ${{ runner.os }}-dependencies-${{ hashFiles('**/composer.lock') }}

#### Closing Summary
and Common Mistakes/Follow-Ups

By caching Git dependencies and submodules, you can significantly speed up your CI builds. However, ensure that cache keys are unique to avoid conflicts. Additionally, always test the cache validity by updating cache key logic when needed.

#### Common mistakes
include:

Not considering large files in LFS.
Failing to properly manage submodules.
Ignoring potential cache key collisions.

For follow-ups, consider advanced strategies like using a local caching service or integrating with container orchestration tools for more robust solutions.

**Comparison:**

```
Cache Mechanism	Description	Implementation
Git LFS	For large files outside main history.	actions/checkout@v2 with fetch-depth set to 0.
Submodules	Clones and manages external repo dependencies.	Use git submodule update --init --recursive.
GitHub Actions	Caches project directories or specific paths.	Utilize actions/cache@v2.
```

**Code:**

```yaml
# Example GitHub Actions workflow
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2
      with:
        fetch-depth: 0

    - name: Initialize submodules
      run: git submodule update --init --recursive

    - name: Cache dependencies
      id: cache_dependencies
      uses: actions/cache@v2
      with:
        path: vendor
        key: ${{ runner.os }}-dependencies-${{ hashFiles('**/composer.lock') }}

    - name: Install dependencies
      run: composer install --no-dev --optimize-autoloader

    - name: Run tests
      run: make test
```

**Example input / output:**

```text
# Example of a cache hit and miss
Cache hit: Dependency files not re-downloaded, saved time.
Cache miss: Fetching new dependencies from network, increased build time.
```

**Data:**

```json
{
  "fetch-depth": 0,
  "path": "vendor",
  "key": "${{ runner.os }}-dependencies-${{ hashFiles('**/composer.lock') }}"
}
```

**Flow:**

1. Yes
2. No
3. Begin CI Run
4. Is LFS Needed?
5. Use Git LFS
6. Initialize Submodules
7. Cache Project DirectoriesUsing GitHub Actions
8. Install Dependencies and RunTests
9. End CI Run

