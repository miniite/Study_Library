---
type: "practice"
difficulty: "Deep"
---

# You need to wire DORA metrics from your CI and deploy tooling; which metric would you chase first for a team that deploys weekly with a high change-failure rate, and how would you instrument it without gaming?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ DORA Metrics](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/DORA_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the DORA metrics challenge in your CI/CD pipeline, we need to prioritize addressing the "Lead Time for Changes" metric first. This is crucial because a high change-failure rate indicates that changes are not being deployed effectively, which can lead to waste and inefficiency. By focusing on reducing the time it takes for code to move from the development environment to production, you can improve overall deployment stability and reduce failures.

Let's break down the problem step by step:

#### Triage

The question is asking about prioritizing DORA metrics in a context where deployments are weekly but have a high change-failure rate. The key metric here is "Lead Time for Changes," as it directly influences the success of deployments. We need to instrument this metric without gaming, ensuring that the improvement efforts contribute genuinely to reducing failure rates.

#### Assumptions
The team deploys changes weekly.
There is currently a high change-failure rate, indicating inefficiencies in the deployment process.
We aim to improve both lead time and reduce failures.

#### First-principles Solution

Identify the Current Lead Time for Changes:

Collect timestamps when code changes are made in the development environment (e.g., Git commit times).
Track when these changes are deployed into production or a pre-production environment.
Calculate the time difference between these two points to determine the lead time.

Instrumentation Without Gaming:

Use automated tools to capture timestamps accurately.
Ensure that developers do not manually manipulate timestamps or deploy times.
Implement a centralized logging system to record all deployment events, which can be cross-referenced with commit logs.

Continuous Monitoring and Feedback:

Set up alerts and dashboards to visualize the lead time for changes over time.
Integrate this metric into regular team retrospectives and process improvements.

Process Improvements:

Analyze bottlenecks in the CI/CD pipeline to identify areas where lead times can be reduced (e.g., automated testing, infrastructure setup).
Implement strategies such as parallelizing builds, optimizing test coverage, or reducing manual steps.
Ensure that all changes go through a consistent and automated deployment process.

#### Closing Summary

By focusing on reducing the "Lead Time for Changes," we can directly address the inefficiencies in the deployment pipeline. This requires accurate instrumentation to avoid gaming the metrics and continuous monitoring to drive improvement. The ultimate goal is not just to measure lead time but also to improve the overall reliability of deployments.

#### Common Mistakes/Follow-ups
Gaming Metrics: Ensure that developers do not manipulate timestamps or deployment times.
Ignoring False Positives: Be cautious about changes that fail tests and are rolled back; these should not inflate the lead time metric.
Overcomplicating the Process: Avoid adding unnecessary complexity to the pipeline just for the sake of reducing lead times. Focus on practical improvements.

**Comparison:**

```
Commit Timestamp	Deployment Timestamp	Lead Time
1634082598	January 2, 2021, 1 PM	24 hours
```

**Code:**

```python
# Example of a simple script to calculate Lead Time for Changes
import os
from datetime import datetime

def get_commit_timestamp(repo_path):
    # Assume we use Git as our version control system
    # This is a placeholder function; actual implementation would involve Git commands
    latest_commit_time = os.popen(f"git --work-tree={repo_path} --git-dir={os.path.join(repo_path, '.git')} log -1 --format=%ct").read().strip()
    return datetime.fromtimestamp(int(latest_commit_time))

def get_deployment_timestamp(deploy_env):
    # This function should fetch the timestamp of when a change was deployed
    # For simplicity, let's assume it returns the current time as an example
    return datetime.now()

commit_time = get_commit_timestamp("/path/to/repository")
deployment_time = get_deployment_timestamp("pre-production")

lead_time = deployment_time - commit_time
print(f"Lead Time for Changes: {lead_time.total_seconds()} seconds")
```

**Example input / output:**

```

Input:

Repository path: /path/to/repository
Deployment environment: pre-production

Output:

PLAINTEXT
Copy
Lead Time for Changes: 3600 seconds

Dry-run Example:

Commit timestamp: 1634082598 (January 1, 2021)
Deployment time: January 2, 2021, at 1 PM
Lead Time for Changes: 24 hours
```

**Data:**

```json
{
  "repository_path": "/path/to/repository",
  "commit_timestamp": 1634082598,
  "deployment_environment": "pre-production"
}
```

**Flow:**

1. Initial Commit
2. CI/CD Pipeline Starts
3. Automated Testing
4. Build and Package Artifacts
5. Determine DeploymentEnvironment
6. Manual or Automated Deployto Pre-Prod
7. Monitor in Production
8. Post-Deployment Monitoring
9. Log Changes and Timestamps
10. Calculate Lead Time
11. Feedback Loop for ProcessImprovement

