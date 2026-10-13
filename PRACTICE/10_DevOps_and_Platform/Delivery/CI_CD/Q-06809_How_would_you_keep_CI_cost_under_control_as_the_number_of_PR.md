---
type: "practice"
difficulty: "Intermediate"
---

# How would you keep CI cost under control as the number of PRs and runs grows?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ CI Cost Optimization](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_Cost_Optimization/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To keep Continuous Integration (CI) costs under control as the number of Pull Requests (PRs) and builds grow, we need to focus on several key areas: optimizing build processes, leveraging caching mechanisms, implementing intelligent trigger strategies, and utilizing cloud-native services for cost-effective scaling. This approach ensures that our CI pipeline remains efficient and responsive even as it handles an increasing volume of PRs.

**Comparison:**

```
Optimization	Description
Caching	Store compiled code, dependencies to reduce repeated work.
Parallelism	Run multiple builds in parallel if infrastructure allows.
Scheduling	Prioritize critical branches or PRs based on urgency.
Cleanup	Automatically clean up old artifacts and logs to save storage.
```

**Code:**

```python
# Example of a basic CI configuration in GitHub Actions
on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [14.x, 16.x]

    steps:
    - uses: actions/checkout@v2
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v2
      with:
        node-version: ${{ matrix.node-version }}
    - run: npm install
    - run: npm test
```

**Example input / output:**

```text
Input:
- Number of PRs per day: 100
- Average build time: 5 minutes

Expected Output (after implementing optimizations):
- Total runtime per day for all builds: <3 hours (e.g., with efficient scheduling and parallelization)
```

**Data:**

```
Sample configuration for GitHub Actions
YAML
Copy
on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [14.x, 16.x]

    steps:
    - uses: actions/checkout@v2
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v2
      with:
        node-version: ${{ matrix.node-version }}
    - run: npm install
    - run: npm test
```

**Flow:**

1. Yes
2. No
3. Initial Setup
4. Is the build cache available?
5. Caching Enabled
6. Check InfrastructureCapabilities
7. Parallel Builds if Possible
8. Implement Scheduling Rules
9. Automation Cleanup Scripts
10. Monitor and Optimize

