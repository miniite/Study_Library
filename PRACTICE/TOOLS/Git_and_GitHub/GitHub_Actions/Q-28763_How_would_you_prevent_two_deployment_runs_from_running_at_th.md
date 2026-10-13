---
type: "practice"
difficulty: "Deep"
---

# How would you prevent two deployment runs from running at the same time?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Concurrency Control in Workflows](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Concurrency_Control_in_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that two deployment runs do not run simultaneously in GitHub Actions, we need to implement a mechanism to manage concurrent execution. This can be achieved by using a combination of job concurrency settings and custom GitHub Actions or scripts. Let’s break this down step-by-step.

#### Triage

The question is asking for a strategy to prevent multiple deployments from running concurrently, ensuring that only one deployment is active at any given time. The interviewer might be probing your understanding of concurrency control in workflows and how to implement such controls effectively using GitHub Actions.

Key Assumptions:
We are dealing with a multi-step deployment process.
Deployments can start frequently (e.g., multiple times per day).
Concurrency needs to be managed across different branches or environments.

#### First-Principles Solution

Understanding Job Concurrency: GitHub Actions allows you to control the concurrency of jobs within a workflow using the concurrency keyword. This can help in managing how many instances of a job are allowed to run simultaneously.

Implementing Lock Mechanism: A simple and effective way is to use a lock file or a custom script that checks for existing deployments before starting a new one. If a deployment is already running, the workflow should skip further steps until the previous deployment completes.

Workflow Configuration Example:

YAML
name: Deployment

on:
  push:
    branches:
      - main
  schedule:
    - cron: '0 8 * * *' # Runs every day at 8 AM

jobs:
  deploy:
    runs-on: ubuntu-latest
    concurrency:
      group: ${{ github.ref }}
      cancel-in-progress: true

    steps:
    - name: Check Deployment Lock
      run: |
        LOCK_FILE_PATH=".deploy.lock"
        if [ -f "$LOCK_FILE_PATH" ]; then
          echo "A deployment is already running. Exiting."
          exit 0
        fi
      id: check_lock

    - name: Acquire Lock
      run: |
        touch .deploy.lock
      if: steps.check_lock.outputs['exit-code'] == 0

    - name: Deploy Application
      # Your deployment steps here
      if: steps.acquire_lock.result == 'success'

    - name: Release Lock
      run: rm -f .deploy.lock
      if: always()

#### Closing Summary
and Common Mistakes:

Key Takeaways:

Use concurrency controls to limit the number of running jobs.
Implement a lock mechanism to prevent overlapping deployments.

Common Pitfalls:

Overlooking concurrency settings, which can lead to unintended parallel executions.
Failing to clean up after locks are released, potentially leaving stale files or locks.
Triaging Further:

If the interviewer delves deeper, they might ask about more advanced locking mechanisms like using a database or external services (e.g., AWS SNS/SQS) for managing deployment locks. This would be more complex but offers robustness in distributed environments.

**Code:**

```yaml
name: Deployment

on:
  push:
    branches:
      - main
  schedule:
    - cron: '0 8 * * *' # Runs every day at 8 AM

jobs:
  deploy:
    runs-on: ubuntu-latest
    concurrency:
      group: ${{ github.ref }}
      cancel-in-progress: true

    steps:
    - name: Check Deployment Lock
      run: |
        LOCK_FILE_PATH=".deploy.lock"
        if [ -f "$LOCK_FILE_PATH" ]; then
          echo "A deployment is already running. Exiting."
          exit 0
        fi
      id: check_lock

    - name: Acquire Lock
      run: |
        touch .deploy.lock
      if: steps.check_lock.outputs['exit-code'] == 0

    - name: Deploy Application
      # Your deployment steps here
      if: steps.acquire_lock.result == 'success'

    - name: Release Lock
      run: rm -f .deploy.lock
      if: always()
```

**Flow:**

1. Lock Present
2. No Lock
3. Triggered Deployment
4. Check Lock
5. Skip
6. End
7. Aquire Lock
8. Deploy Application
9. Release Lock
10. End

