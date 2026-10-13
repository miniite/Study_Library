---
type: "practice"
difficulty: "Intermediate"
---

# Where would you look for logs when a Hadoop job fails and what kind of information do they give you?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ YARN Troubleshooting](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/YARN_Troubleshooting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a Hadoop job fails, looking into the appropriate log directories is crucial to diagnose the issue. The logs provide detailed insights into the execution flow, errors, and performance metrics. For a fresher or a mid-level engineer, understanding where these logs are stored and what information they contain is essential.

Let's break down the process step by step:

Understanding Log Locations: Hadoop stores logs in specific directories based on the type of service or component involved.
Identifying the Logs to Check: Different components like MapReduce tasks, NameNode, DataNode, and task trackers generate different types of logs.
Interpreting the Information: The logs contain critical details about the job's execution, including error messages, stack traces, resource usage, and more.

#### Triage
What is Being Probed

The interviewer wants to understand your familiarity with Hadoop’s architecture and logging mechanisms. They are probing for your knowledge of where to look first when a job fails, as well as what information you would expect from the logs.

#### Assumptions
and Reasoning
Hadoop Version: Assume it's a standard Hadoop 2.x or later version.
Cluster Environment: Consider both single-node and multi-node cluster setups.
Log Types: Logs can be categorized into job-related, node-specific, and system-level logs.
Job-Related Logs (MR Application)
JobTracker Logs:
Located in hadoop/mapreduce/log/<jobtracker-hostname>/
Contains overall job metrics and status updates.
TaskTracker/TaskLogs:
Located in hadoop/mapreduce/log/<tasktracker-hostname>/logs/attempt_*_map*_*_000001
Detailed logs for each task, including input/output splits, shuffle processes, and errors.
Node-Specific Logs
NameNode Logs:
Located in hadoop/hdfs/logs/nm-<nodemanager-hostname>/
Contains data node logs which can provide insights into storage operations.
DataNode Logs:
Located in hadoop/hdfs/data/current/
Details on block transfers, replication, and other storage-related activities.
System-Level Logs
HDFS Namenode Web UI Logs:
Available at http://namenode:50070/ (check logs under the "Log Browser" section)
General Hadoop Logs:
Found in hadoop/logs/
Includes system-wide error messages and warnings.
Step-by-Step Solution
Identify the Job ID: First, identify the job ID from the error message or Hadoop UI.
Navigate to TaskTracker Logs: Use the job ID to navigate to the corresponding task tracker logs.
Review the Log Files:
Check for any stderr and stdout files which might contain specific errors.
Look for stack traces, exceptions, and resource consumption metrics.
Analyze NameNode and DataNode Logs: If you suspect issues with data handling or storage, look at relevant logs from the NameNode and DataNodes.

#### Closing Summary

By systematically checking these log locations, one can gain a comprehensive understanding of what went wrong during the job execution. This approach ensures that both application-level and system-wide issues are addressed effectively.

#### Common Mistakes/Follow-Ups
Mistake 1: Ignoring the stderr output for critical error messages.
Mistake 2: Failing to check the relevant task logs before looking at general Hadoop logs.

#### Follow-Up
How would you handle a job that fails due to network issues? What steps would you take differently?

**Code:**

```bash
# Example of navigating to Hadoop logs directory
hadoop fs -ls /hadoop/mapreduce/log/tasktracker-hostname/logs/attempt_*_map*_*_000001
```

