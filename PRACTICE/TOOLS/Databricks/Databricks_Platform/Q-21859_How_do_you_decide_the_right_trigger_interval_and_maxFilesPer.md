---
type: "practice"
difficulty: "Deep"
---

# How do you decide the right trigger interval and maxFilesPerTrigger for a streaming ingest that must keep p95 end-to-end latency under 2 minutes without over-provisioning?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Streaming Ingest Tuning](<../../../../TOOLS/Databricks/Databricks_Platform/Streaming_Ingest_Tuning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of setting the right triggerInterval and maxFilesPerTrigger in Databricks for a streaming ingest that must maintain p95 end-to-end latency under 2 minutes, we need to consider several factors including data volume, processing speed, resource allocation, and the nature of the stream. The goal is to balance between responsiveness and cost efficiency.

This question primarily tests your understanding of streaming concepts, Databricks configuration parameters, and performance optimization strategies. We'll approach this from first principles, ensuring a deep understanding of each parameter and its implications on latency and throughput.

Firstly, let's define triggerInterval and maxFilesPerTrigger. The triggerInterval is the time interval after which an action (like writing to storage) can be triggered in Spark Structured Streaming. maxFilesPerTrigger, if enabled, limits the number of input files processed by a trigger.

Triaging the Problem
Understanding Requirements: We need p95 end-to-end latency under 2 minutes. This means we must ensure that 95% of our data has a processing time less than 2 minutes from ingestion to final output.
Identifying Key Parameters:
triggerInterval: Determines the frequency at which Spark processes new data and triggers actions.
maxFilesPerTrigger: Limits the number of files processed in each trigger, affecting both latency and throughput.

#### First Principles Solution
Determine Data Volume: Estimate the average file size and arrival rate of incoming data. For example, if you receive 100MB files every minute, understand how this volume impacts processing time.
Processing Speed: Assess the performance of your Spark jobs. How long does it take to process a single batch of data? This is critical in determining triggerInterval.
Resource Allocation: Evaluate the resources available (CPU, memory) and ensure they are properly configured for efficient processing.
Step-by-Step Solution

Calculate Processing Time:

Suppose each file takes 5 seconds to process.
If you have a trigger interval of 60 seconds, you can batch up to 12 files in one go (12 * 5 = 60 seconds).
For p95 latency under 2 minutes, we need to ensure that the longest processing time is within this limit.

Determine triggerInterval:

Given our example where each file takes 5 seconds to process, a trigger interval of 60 seconds (10 files) seems reasonable.
However, if you find that p95 latency consistently exceeds 2 minutes, you may need to reduce the triggerInterval.

Set maxFilesPerTrigger:

If reducing the triggerInterval does not significantly improve p95 latency, consider setting a lower value for maxFilesPerTrigger.
For instance, if processing 10 files per trigger still results in higher than acceptable latency, try setting maxFilesPerTrigger to 6 or 8.

Monitor and Adjust:

Use monitoring tools to track p95 latency and adjust parameters as needed.
Ensure that your Spark jobs are optimized for the number of tasks and resources available.

#### Closing Summary

By carefully considering the data volume, processing speed, and resource allocation, you can set triggerInterval and maxFilesPerTrigger effectively. The key is to balance responsiveness (low latency) with cost efficiency (not over-provisioning). Continuous monitoring and iterative adjustments will ensure that your streaming ingest meets the required SLA.

#### Common Mistakes/Follow-Ups
Over-provisioning: Setting overly short triggerInterval can lead to excessive resource utilization without significant improvement in latency.
Under-utilization: Ignoring the processing time for each file can result in suboptimal maxFilesPerTrigger, leading to unnecessary delays.
Lack of Monitoring: Failing to continuously monitor and adjust parameters based on real-time data can cause performance issues.

