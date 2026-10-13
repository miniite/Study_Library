---
type: "practice"
difficulty: "Expert"
---

# You need to bisect which of 200 commits introduced a slow memory leak in the inference server, but each candidate requires deploying and soaking under load. How would you script git bisect run around a deploy-and-observe loop and avoid false positives?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Git Bisect](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Bisect.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we'll leverage git bisect to efficiently identify which commit introduced the memory leak by deploying candidate commits under load in an automated fashion. This method minimizes manual effort and avoids false positives through careful setup and scripting.

First, let's understand what git bisect is doing: it narrows down the problem by repeatedly dividing the range of commits between a known "bad" commit (where the leak occurs) and a known "good" commit (where no memory issues are observed). The goal is to find the first bad commit.

#### Triaging the Question

The key points:

We have 200 commits that potentially introduced the memory leak.
Each commit requires deploying and observing under load, which implies significant overhead.
The task is to minimize false positives by ensuring accurate identification of the culprit.

#### Assumptions
There is one commit (or a small number) between our known "good" and "bad" commits that introduced the issue.
The deployment process can be automated.
We have access to monitoring tools to observe memory usage under load.

#### First-Principles Solution
Step 1: Setting Up git bisect

Start by identifying a commit we know is good (no memory leak) and a bad one (known issue).

BASH
git bisect start [good-commit-hash] [bad-commit-hash]
Step 2: Automating the Deployment and Observation Process

We'll write a script that performs the following steps:

Clone the repository: Ensure you have the correct branch checked out.
Switch to the current bisect commit.
Deploy the server: Use Docker or another deployment method.
Monitor memory usage under load: Run a load test and record memory metrics.

Here's an example script:

BASH
#!/bin/bash

# Load configuration variables from environment or file
source ./config.sh

function deploy_and_monitor() {
  git checkout $GIT_COMMIT
  # Deploy the server using Docker or other method
  docker-compose up -d
  sleep 5m  # Allow time for the server to start and reach a steady state

  # Run load test and monitor memory usage
  python3 ./load_test.py --duration=1h | while read -r line; do
    # Parse and record memory usage metrics (e.g., using `top` or custom logging)
    mem_usage=$(echo $line | grep "Mem" | awk '{print $2}')
    echo "$mem_usage"
  done
}

while [ true ];
do
  if deploy_and_monitor; then
    # If no memory issues, mark as good and continue bisecting
    git bisect good
  else
    # If a memory issue is detected, mark as bad and stop bisecting
    git bisect bad
    break
  fi
done

git bisect reset  # Clean up after finding the bad commit
Step 3: Handling False Positives

To avoid false positives:

Ensure that load tests are representative of real-world scenarios.
Use multiple metrics (e.g., memory, CPU, network) to confirm the presence of a leak.
Implement a threshold for acceptable memory usage before marking a commit as "bad".

#### Closing Summary
and Common Mistakes
False Positives: Avoiding false positives is crucial; ensure your monitoring tools are robust and that you have multiple metrics for verification.
Deployment Time: Account for the time it takes to deploy and stabilize the server, which can vary depending on network conditions or hardware.
Resource Management: Optimize resource usage during load testing to avoid overwhelming your infrastructure.

By following this script and ensuring thorough monitoring, we can efficiently identify the commit that introduced the memory leak while minimizing deployment overhead.

Common Follow-Ups
What if there are multiple issues?
How do you handle different environments (development vs production)?
Are there any additional tools or configurations needed for this process?

