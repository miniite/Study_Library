---
type: "practice"
difficulty: "Deep"
---

# Your runner CI minutes bill spiked this month. Walk me through how you would attribute cost and cut it.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Docker Layer Caching](<../../../../TOOLS/GitLab/GitLab_CI/Docker_Layer_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the spike in GitLab CI minutes, we need to diagnose where the costs are coming from and implement strategies to reduce them. This process involves understanding our current usage patterns, identifying resource-intensive jobs, and optimizing the pipeline configuration.

Firstly, let's clarify what we're dealing with. The cost is likely due to increased execution time of GitLab CI jobs, which translates to higher runner minutes. We need to investigate if there are specific jobs or stages that have suddenly become more expensive.

#### Triage
What’s Being Probed?

The interviewer is probing your ability to:

Analyze and diagnose the root cause of the cost increase.
Understand resource management in CI/CD pipelines.
Optimize pipeline configurations for efficiency.
Step-by-Step Solution
1. Gather Data

We start by collecting data on our GitLab project's CI usage. This includes:

The total number of job executions this month compared to the previous months.
The duration and frequency of specific jobs.
Runner utilization metrics like CPU, memory, and disk I/O.

To gather this information, we can use GitLab’s built-in reports or export data from the runner logs and CI/CD activity API.

2. Identify Resource-Intensive Jobs

Next, we identify which jobs are consuming more resources:

Long-running jobs: Jobs that take longer than usual.
High-concurrency jobs: Jobs that run simultaneously, potentially causing bottlenecks.
Large builds: Builds that involve large repositories or complex processes.

We can use GitLab’s CI/CD activity report to filter and sort jobs by execution time. For example:

MARKDOWN
| Job Name       | Execution Time (avg) | Runs this month |
|----------------|----------------------|-----------------|
| JobA           | 120 seconds          | 50              |
| JobB           | 360 seconds          | 20              |
| JobC           | 840 seconds          | 10              |
3. Analyze Runner Utilization

We analyze runner utilization to see if there are any bottlenecks:

CPU and Memory: Are runners under heavy load?
Disk I/O: Is disk usage causing delays?

This can be done by examining the runner logs or using monitoring tools integrated with GitLab.

4. Optimize Pipeline Configuration

Based on the analysis, we make the following optimizations:

Parallelize Jobs: If jobs are not dependent on each other, run them in parallel to reduce total execution time.
Optimize Job Scripts: Review and optimize job scripts for efficiency:
Use caching where applicable.
Optimize Docker images by removing unnecessary layers.
Upgrade Runner Resources: If certain runners are consistently overloaded, consider upgrading their resources or adding more runners.
5. Implement Cost Controls

Finally, we implement cost controls to ensure future spikes are managed:

Set up alerting on high resource usage or long-running jobs.
Monitor runner performance regularly and adjust as needed.

#### Closing Summary
+ Common Mistakes/Follow-Ups

To summarize, the key steps are:

Collect data on CI job execution times and resource utilization.
Identify and prioritize resource-intensive jobs.
Optimize pipeline configurations to reduce execution time.
Implement cost controls to manage future spikes.

#### Common mistakes
to avoid include not thoroughly analyzing the root cause or focusing solely on short-term optimizations without considering long-term strategies.

#### Follow-ups
might include:

How you would automate the process of identifying and optimizing jobs?
What tools or scripts do you use for monitoring CI/CD pipelines?

**Comparison:**

```
tableJobUsage {
    title "Job Usage Summary"
    row Number, Name, Avg Time (s), Runs This Month
    colRef 0
    3 120 50
    3 360 20
    3 840 10
}
```

**Code:**

```python
# Example script to gather CI usage data
import gitlab

gl = gitlab.Gitlab('https://gitlab.example.com', private_token='your_private_token')

# Get all projects and their CI usage
projects = gl.projects.list(all=True)
for project in projects:
    print(project.name, project.namespace['name'])
```

**Example input / output:**

| Job Name       | Avg Time (s) | Runs This Month |
|----------------|--------------|-----------------|
| JobA           | 120          | 50              |
| JobB           | 360          | 20              |
| JobC           | 840          | 10              |

**Data:**

```json
{
  "jobs": [
    {
      "name": "JobA",
      "avg_time": 120,
      "runs_this_month": 50
    },
    {
      "name": "JobB",
      "avg_time": 360,
      "runs_this_month": 20
    },
    {
      "name": "JobC",
      "avg_time": 840,
      "runs_this_month": 10
    }
  ]
}
```

**Flow:**

1. Collect Data
2. Identify Resource-IntensiveJobs
3. Analyze Runner Utilization
4. Optimize PipelineConfiguration
5. Implement Cost Controls
6. Monitor and Adjust

