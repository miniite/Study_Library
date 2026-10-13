---
type: "practice"
difficulty: "Basic"
---

# How do you make one GitHub Actions job wait for another to finish using needs?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reducing Actions Costs](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reducing_Actions_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that one GitHub Actions workflow waits for another to complete, we can leverage the needs keyword in our workflows. This ensures that a subsequent job only runs after its prerequisites have successfully completed. Let's dive into how this works and explore some key points.

Firstly, it’s important to understand what needs are: they provide a way to specify dependencies between jobs in a single workflow or across different workflows. By using needs, we can create a more robust pipeline where certain steps must wait for others before executing.

Here's a step-by-step guide on how to implement this:

Define the First Workflow: Create a GitHub Actions workflow that performs some initial task.
Specify Dependencies in the Second Workflow: Use the needs keyword in the second workflow to define which jobs it needs to wait for.
Run the Workflows Sequentially or Parallel: Depending on your setup, you might run these workflows sequentially or as part of a larger pipeline.

Let's look at an example where we have two workflows: one that runs some tests and another that deploys code based on test results.

#### Example
Workflow Setup
First Workflow (Tests)
YAML
name: Run Tests
on:
  push:
    branches: [ main ]
jobs:
  run-tests:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: npm test

Second Workflow (Deployment)
YAML
name: Deploy Code
on:
  push:
    branches: [ main ]
jobs:
  deploy-code:
    needs: run-tests
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      - name: Install dependencies
        run: npm install
      - name: Deploy to production
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
        run: |
          echo "Deploying code..."
          # Run your deployment logic here

In this setup, the deploy-code job in the second workflow is marked with needs: run-tests. This means that it will only start running after the run-tests job has completed successfully.

#### Common Mistakes
and Follow-Ups
Not Using Needs Correctly: Some might mistakenly use uses instead of needs, which would not enforce dependency execution.
Forgetting to Define Prerequisites: Ensure that all necessary steps are defined as prerequisites in the workflow configuration.
Handling Failures Gracefully: Consider adding error handling and retry mechanisms for critical jobs.

By correctly implementing dependencies with needs, you can ensure that your GitHub Actions workflows run in a controlled and predictable manner, enhancing the reliability of your CI/CD pipeline.

**Comparison:**

```
Workflow	Event	Job Name	Needs	Steps
Run Tests	push (main)	run-tests	-	Checkout, install dependencies, run tests
Deploy	push (main)	deploy-code	run-tests	Checkout, install dependencies, deploy to production with secret key
```

**Code:**

```yaml
# Example Workflow 1 - Run Tests
name: Run Tests
on:
  push:
    branches: [ main ]
jobs:
  run-tests:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: npm test

# Example Workflow 2 - Deploy Code
name: Deploy Code
on:
  push:
    branches: [ main ]
jobs:
  deploy-code:
    needs: run-tests
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      - name: Install dependencies
        run: npm install
      - name: Deploy to production
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
        run: |
          echo "Deploying code..."
          # Run your deployment logic here
```

**Example input / output:**

```

Input:

YAML
Copy
# Workflow with Needs Example
name: Run Tests
on:
  push:
    branches: [ main ]
jobs:
  run-tests:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: npm test

name: Deploy Code
on:
  push:
    branches: [ main ]
jobs:
  deploy-code:
    needs: run-tests
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      - name: Install dependencies
        run: npm install
      - name: Deploy to production
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
        run: |
          echo "Deploying code..."

Output:

PLAINTEXT
Copy
run-tests job completes successfully.

deploy-code job starts after run-tests has completed.
```

**Data:**

```json
{
  "workflow1": {
    "name": "Run Tests",
    "triggers": [
      { "event": "push", "branches": [ "main" ] }
    ],
    "jobs": {
      "run-tests": {
        "runs-on": "ubuntu-latest",
        "steps": [
          { "name": "Checkout repository", "uses": "actions/checkout@v2" },
          { "name": "Install dependencies", "run": "npm install" },
          { "name": "Run tests", "run": "npm test" }
        ]
      }
    }
  },
  "workflow2": {
    "name": "Deploy Code",
    "triggers": [
      { "event": "push", "branches": [ "main" ] }
    ],
    "jobs": {
      "deploy-code": {
        "needs": "run-tests",
        "runs-on": "ubuntu-latest",
        "steps": [
          { "name": "Checkout repository", "uses": "actions/checkout@v2" },
          { "name": "Install dependencies", "run": "npm install" },
          { "name": "Deploy to production", "env": { "DEPLOY_KEY": "${{ secrets.DEPLOY_KEY }}" }, "run": "echo 'Deploying code...'" }
        ]
      }
    }
  }
}
```

**Flow:**

1. Run Tests
2. Checkout
3. Install Dependencies
4. Run Tests
5. Deploy Code
6. Needs: Run Tests
7. Checkout
8. Install Dependencies
9. Deploy to Production withSecret Key

