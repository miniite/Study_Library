---
type: "practice"
difficulty: "Intermediate"
---

# What is caching in GitHub Actions and how does actions/cache speed up dependency installs?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ CI Cache Correctness](<../../../../TOOLS/Git_and_GitHub/Git_Core/CI_Cache_Correctness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
In this answer, we'll discuss what caching in GitHub Actions is, its purpose, and how actions/cache can significantly speed up dependency installations. We'll also explore different scenarios where caching proves beneficial.

#### Triage
The question asks about the concept of caching within GitHub Actions specifically with actions/cache. This involves understanding:

What caching is.
How caching works in general, especially in the context of CI/CD pipelines.
The specific implementation and benefits of using actions/cache.

#### First Principles Solution

What Caching Is: Caching is a technique used to store data or results from previous computations so that those same results can be retrieved quickly if the same inputs occur again, rather than recalculating them. In the context of GitHub Actions, caching helps by storing previously installed dependencies and reusing them when necessary.

How actions/cache Works:

Purpose: The actions/cache action is designed to cache package managers' state (e.g., npm, yarn, pip) or other outputs from previous jobs.
Process: When you use actions/cache, the initial run stores a snapshot of your dependencies. On subsequent runs, if the same dependencies are needed and haven't changed, GitHub Actions retrieves these cached dependencies rather than reinstalling them.

Benefits:

Speeds Up CI/CD Pipelines: By reusing cached data, dependency installations become much faster.
Reduces Costs: Frequent builds or deployments can be costly due to repeated downloads and installs. Caching mitigates these costs by avoiding unnecessary network requests.
Improved Reliability: If there are network issues or timeouts during the initial installation, subsequent runs using cached data avoid these problems.

#### Example
Use Case:

YAML
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Node.js cache
        id: cache-step
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-

      - name: Install dependencies
        run: npm install --only=production

      # Rest of the workflow...

#### Common Mistakes
and Follow-Ups:

Misconfiguring Cache Keys: A common mistake is not configuring cache keys correctly, which can lead to stale data being reused.
Ignoring Non-Reliable Data Sources: Dependencies should be reinstalled if there are changes in the source code or other non-reliable factors.

#### Closing Summary
and Common Mistakes/Follow-Ups: Understanding caching in GitHub Actions is crucial for optimizing CI/CD pipelines. The actions/cache action provides a robust way to cache dependencies, leading to faster builds and reduced costs. However, proper configuration of cache keys and ensuring that cache integrity isn't compromised are essential.

**Code:**

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Node.js cache
        id: cache-step
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-

      - name: Install dependencies
        run: npm install --only=production

      # Rest of the workflow...
```

