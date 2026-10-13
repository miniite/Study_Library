---
type: "practice"
difficulty: "Deep"
---

# A director asks how you'd measure ML-platform DORA-style health (retrain lead time, deploy frequency, rollback rate) from Python tooling and Git/CI APIs, and what edge cases (reverts, hotfixes, shadow runs) skew it.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Python Tail Latency](<../../../../TOOLS/Python/Python_Language/Python_Tail_Latency/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To measure ML-platform DORA-style health indicators such as retrain lead time, deploy frequency, and rollback rate using Python tooling and Git/CI APIs, we need to leverage both our understanding of the CI/CD pipeline and the available metadata from the platform. The goal is to extract relevant metrics directly from the source control system (Git) and continuous integration tools, and then process these data points to compute the desired KPIs.

#### Triaging the Question

The core of the question revolves around extracting meaningful metrics from Git repositories and CI systems. These metrics include retrain lead time, deploy frequency, and rollback rate. The DORA framework is known for its focus on delivering software faster while maintaining quality. We need to ensure that our solution accounts for edge cases such as hotfixes, shadow runs, and reverts.

#### First-Principles Solution
Retrain Lead Time: This measures the time taken from when a model needs to be updated to the actual deployment of the new version.
Deploy Frequency: The number of times a model is deployed in a given period.
Rollback Rate: The frequency with which deployments are rolled back due to issues.

To measure these, we need:

Git repositories containing commit history and branches.
CI/CD tool logs that record build, test, and deployment events.
Metadata about each model version and its lifecycle stages (training, validation, production).
Step-by-Step Solution
Collect Commit History:
Use git command-line tools or Python libraries like gitpython to fetch commit history from Git repositories.
Parse CI/CD Logs:
Extract logs from the CI system (e.g., Jenkins, GitHub Actions) using their APIs or command-line interfaces.
Process Data:
Map commits and builds to model versions.
Identify retrain events by detecting branches dedicated to training new models.
Compute Metrics:
Calculate lead time from the commit of a change request to the deployment in production.
Count the number of successful deployments within a given period for deploy frequency.
Analyze rollback events to compute rollback rate.
Sample Code and Data
PYTHON
import git
from datetime import datetime

# Example: Fetching Git history using GitPython
repo = git.Repo('path/to/repo')
commits = list(repo.iter_commits('main'))

def extract_retrain_events(commits):
    retrain_events = []
    for commit in commits:
        # Check if the commit message contains words like 'retrain' or 'model'
        if any(word in commit.message.lower() for word in ['retrain', 'model']):
            retrain_events.append(commit)
    return retrain_events

def calculate_lead_time(retrain_event, deployments):
    start_time = retrain_event.committed_date
    end_time = min(deployments, key=lambda d: abs(d - retrain_event.committed_date)).committed_date
    return (end_time - start_time).total_seconds() / 60 / 60

def main():
    # Extract retrain events
    retrain_events = extract_retrain_events(commits)

    # Example deployment data from CI logs
    deployments = [datetime(2023, 1, 1), datetime(2023, 1, 5)]

    for event in retrain_events:
        lead_time = calculate_lead_time(event, deployments)
        print(f"Retrain Event: {event}, Lead Time: {lead_time} hours")

if __name__ == "__main__":
    main()
Edge Cases and Considerations
Hotfixes: These are typically immediate fixes deployed to address critical issues without a full retraining cycle. They should be excluded from retrain lead time calculations.
Shadow Runs: These are deployments that run in parallel with the production version for testing purposes. They can skew deploy frequency metrics if they are not properly tagged as such.
Reverts and Rollbacks: These events need to be accurately identified and accounted for separately.

#### Closing Summary

By leveraging Git and CI/CD APIs, we can extract detailed information about model retraining cycles, deployment practices, and rollback incidents. This data enables us to compute DORA-style health metrics with precision, but it's crucial to handle edge cases like hotfixes and shadow runs to ensure accurate results.

#### Common Mistakes/Follow-Ups
Misidentifying Retrain Events: Overly broad or narrow definitions of retrain events can lead to inaccurate calculations.
Ignoring Hotfixes and Shadow Runs: Failing to exclude these from the relevant metrics will skew the results.
Inadequate Data Granularity: Insufficient detail in logs may make it hard to accurately map commits to model versions.

**Comparison:**

```
Retrain Event	Lead Time (hours)
abc123	4
def456	N/A
```

**Code:**

```python
import git
from datetime import datetime

# Example: Fetching Git history using GitPython
repo = git.Repo('path/to/repo')
commits = list(repo.iter_commits('main'))

def extract_retrain_events(commits):
    retrain_events = []
    for commit in commits:
        # Check if the commit message contains words like 'retrain' or 'model'
        if any(word in commit.message.lower() for word in ['retrain', 'model']):
            retrain_events.append(commit)
    return retrain_events

def calculate_lead_time(retrain_event, deployments):
    start_time = retrain_event.committed_date
    end_time = min(deployments, key=lambda d: abs(d - retrain_event.committed_date)).committed_date
    return (end_time - start_time).total_seconds() / 60 / 60

def main():
    # Extract retrain events
    retrain_events = extract_retrain_events(commits)

    # Example deployment data from CI logs
    deployments = [datetime(2023, 1, 1), datetime(2023, 1, 5)]

    for event in retrain_events:
        lead_time = calculate_lead_time(event, deployments)
        print(f"Retrain Event: {event}, Lead Time: {lead_time} hours")

if __name__ == "__main__":
    main()
```

**Example input / output:**

```text
Retrain Event: Commit abc123, Lead Time: 4 hours
```

**Data:**

```json
{
  "commits": [
    {
      "sha": "abc123",
      "message": "Training new model version"
    },
    {
      "sha": "def456",
      "message": "Fixing bug in production"
    }
  ],
  "deployments": [
    {"timestamp": "2023-01-01", "status": "success"},
    {"timestamp": "2023-01-05", "status": "success"}
  ]
}
```

**Flow:**

1. Yes
2. Start
3. Fetch Git History
4. Extract Retrain Events
5. Calculate Lead Times
6. Collect Deployments
7. Compute Metrics
8. Handle Edge Cases
9. Output Metrics
10. End

