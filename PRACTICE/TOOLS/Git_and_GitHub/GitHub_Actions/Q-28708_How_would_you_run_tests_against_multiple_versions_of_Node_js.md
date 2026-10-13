---
type: "practice"
difficulty: "Intermediate"
---

# How would you run tests against multiple versions of Node.js using a matrix strategy?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Concurrency Control in Workflows](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Concurrency_Control_in_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address running tests against multiple versions of Node.js using a matrix strategy with GitHub Actions, we need to understand that this involves setting up a job in our .yml file to execute tasks across different Node.js version combinations. This is particularly useful for ensuring compatibility and stability across various environments.

#### Triaging the Question

The core of this question revolves around:

Setting Up a Matrix Strategy: Understanding how to define multiple versions of software (in this case, Node.js) that need to be tested.
Running Tests in GitHub Actions: Configuring the .yml file to handle these test runs efficiently.

#### Assumptions
We are using a GitHub Actions workflow file for CI/CD.
The goal is to ensure tests run against multiple versions of Node.js, such as 14.x, 16.x, and 20.x.
Each version should trigger the same set of test steps.

#### First-Principles Solution

Let's start by setting up a GitHub Actions workflow file that uses a matrix strategy to run tests across different Node.js versions. The key is defining a matrix in the jobs section and ensuring each job runs the necessary commands for testing.

Step-by-Step Solution
Define Matrix Strategy: In the jobs section, use the strategy:matrix keyword to define multiple Node.js version combinations.
Set Up Job Steps: For each version defined in the matrix, set up steps within the job to install the Node.js version and run tests.

#### Example
Workflow File
YAML
name: Test with Different Node.js Versions

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  test_nodejs_versions:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [14.x, 16.x, 20.x]
    steps:
    - name: Check out repository
      uses: actions/checkout@v3

    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, setting up a GitHub Actions workflow to run tests across multiple Node.js versions involves defining a matrix strategy and ensuring each job version runs the necessary steps. The common mistake is forgetting to define the matrix properly or missing critical steps like installing dependencies before running tests.

#### Follow-Ups
How do you handle caching dependencies between different Node.js versions?
What are some best practices for managing secrets in your GitHub Actions workflows?

**Code:**

```yaml
name: Test with Different Node.js Versions

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  test_nodejs_versions:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [14.x, 16.x, 20.x]
    steps:
    - name: Check out repository
      uses: actions/checkout@v3

    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test
```

