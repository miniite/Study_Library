---
type: "practice"
difficulty: "Intermediate"
---

# You push a fix but the site still shows the old version. Name two simple things you would check.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Artifact Promotion Across Environments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Artifact_Promotion_Across_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When encountering an issue where a pushed fix is not reflecting in the live environment, as a DevOps engineer with intermediate experience, it's essential to systematically diagnose potential issues from multiple perspectives. The two simple things you would check are:

Environment and Branch Configuration: Ensure that the correct branch and environment are being deployed.
Deployment Status and Logs: Verify if the deployment was successful and examine logs for any errors or issues.

Let’s dive deeper into each of these:

#### Triage
What is Being Probed

The question aims to understand your ability to methodically debug a CI/CD pipeline issue, which involves checking fundamental aspects such as environment setup and deployment status. The interviewer might also want to know if you consider potential misconfigurations or overlooked steps in the deployment process.

#### Assumptions
and First Principles Solution

Environment and Branch Configuration

Confirm that the code changes were committed to the correct branch (e.g., master, main).
Verify that the environment where the fix should be deployed is correctly configured and selected during the CI/CD pipeline.
Ensure that there are no issues with merge conflicts or uncommitted changes that might have affected the build.

Deployment Status and Logs

Check if the deployment process completed successfully by reviewing the CI/CD pipeline logs for any errors or warnings.
Investigate any timeout or failure messages during the deployment stage, which could indicate problems like infrastructure issues or network constraints.
Ensure that the application is correctly scaling in the environment to handle traffic. For example, confirm that there are no pod crashes or health checks failing.
Step-by-Step Solution

Verify Branch and Environment Settings

SH
git status # To check if you are on the correct branch
kubectl config get-contexts # If using Kubernetes, to verify current context (environment)

Review Deployment Pipeline Logs

SH
# For a Jenkins pipeline
curl http://jenkins/job/<job-name>/lastBuild/api/json | jq '.result'

# For GitLab CI/CD
gitlab-ci-multi-runner exec <runner-configuration> shell -- tail -n 100 <job-name>.log

Check Application Status and Logs

SH
kubectl get pods,svc -n <namespace> # To check if the application pods are running
kubectl logs <pod-name> -n <namespace> # For detailed logs of the pod

#### Closing Summary
+ Common Mistakes/Follow-Ups

By systematically checking these two areas—branch and environment configuration, as well as deployment status and logs—you can quickly identify common issues that might prevent a fix from reflecting in the live environment. Some common mistakes to avoid include:

Assuming that the branch is correct without verifying.
Overlooking critical messages in logs due to lack of proper monitoring.
Not considering potential scaling or resource constraints.

For follow-ups, you could discuss additional steps such as checking service discovery and network configurations if necessary, or ensuring that caches are cleared in a production environment. These steps demonstrate your ability to think holistically about the deployment process and troubleshoot effectively.

**Comparison:**

```
Step	Action
1	Verify Git branch and Kubernetes context.
2	Check the CI/CD pipeline logs for errors.
3	Review application pod statuses in Kubernetes.
4	Examine detailed pod logs if deployment issues are found.
```

**Code:**

```
SH
Copy
# Example commands for verification
git status # Git branch check
kubectl config get-contexts # Kubernetes context check
curl http://jenkins/job/<job-name>/lastBuild/api/json | jq '.result' # Jenkins pipeline log check
gitlab-ci-multi-runner exec <runner-configuration> shell -- tail -n 100 <job-name>.log # GitLab CI/CD logs
kubectl get pods,svc -n <namespace> # Kubernetes pod and service status
kubectl logs <pod-name> -n <namespace> # Pod logs for troubleshooting
```

**Example input / output:**

```text
- **Input**: `git status`, `kubectl config get-contexts`, logs from Jenkins/GitLab
- **Output**: Verifications of branch and environment, deployment status, and log contents.
```

**Data:**

```json
{
  "branch": "main",
  "environment": "production",
  "deployment_status": "completed with errors",
  "logs": [
    "Error: deployment failed due to network issues"
  ]
}
```

**Flow:**

1. Yes
2. No
3. Verify Branch and Environment
4. Is the branch correct?
5. Check Deployment Status
6. Switch to Correct Branch
7. Review Logs for Errors
8. Check Application Pods
9. Examine Detailed Pod Logs ifNeeded

