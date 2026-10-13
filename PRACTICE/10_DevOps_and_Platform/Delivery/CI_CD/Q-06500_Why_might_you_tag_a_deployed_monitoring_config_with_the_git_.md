---
type: "practice"
difficulty: "Intermediate"
---

# Why might you tag a deployed monitoring config with the git commit hash, and how does that help during an incident?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Deployment Observability](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deploying monitoring configurations in a CI/CD pipeline, tagging them with the Git commit hash provides critical traceability and versioning information. This practice helps in quickly identifying which code change led to a specific issue during an incident, facilitating faster troubleshooting and resolution.

#### Triage

The question is asking about the benefits of using Git commit hashes as tags for monitoring configurations deployed via CI/CD pipelines. The interviewer is probing your understanding of how version control and deployment processes interact to support incident management.

#### First-Principles Solution
Version Control and Deployment: In a CI/CD pipeline, each code change is associated with a unique Git commit hash. This hash uniquely identifies the exact state of the code at any point in time.
Tagging Monitoring Configurations: By tagging monitoring configurations with these commit hashes, you can ensure that specific versions of your monitoring setup are directly linked to particular code changes. This linkage helps in diagnosing issues by pinpointing exactly which change introduced a problem.
Incident Response: When an incident occurs, being able to quickly look up the associated commit hash allows the team to:
Revert or roll back to previous versions of the monitoring setup if needed.
Verify that the issue is not due to recent changes and may be related to something else.
Understand the exact state of the system at the time of the incident, aiding in accurate root cause analysis.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Tagging with commit hashes ensures that every deployment has a unique identifier, making it easier to track and manage changes. This practice is especially valuable during incidents as it allows for rapid and precise diagnosis of issues by correlating them with specific code changes. Common mistakes include neglecting this tagging process or not maintaining accurate version information in the monitoring configurations.

#### Follow-ups
might include:

Discussing how automated tools can be used to tag and track these hashes.
Exploring scenarios where multiple deployments occur simultaneously.
Addressing potential challenges such as managing large numbers of tags efficiently.

**Comparison:**

```
Incident Time	Event Type	Message
10:00 AM	Deployment	Deployed monitoring config with commit hash abc123def456
11:00 AM	Alert	Monitoring alert triggered by recent changes (commit abc123def456)
```

**Code:**

```python
# Example code snippet for tagging a monitoring config with the commit hash
def apply_monitoring_config(commit_hash, monitoring_config):
    # Apply the monitoring config with its associated commit hash
    print(f"Applying monitoring config with commit hash: {commit_hash}")
```

**Example input / output:**

| Commit Hash | Monitoring Config Version |
|-------------|--------------------------|
| abc123def456 | v1.2.3                   |

**Data:**

```json
{
  "config": {
    "type": "monitoring",
    "version": "v1.2.3",
    "hash": "abc123def456"
  }
}
```

**Flow:**

1. Incident Occurs
2. Identify Affected Configs
3. Retrieve Commit Hashes
4. Determine Root Cause
5. Rollback to Previous Version

