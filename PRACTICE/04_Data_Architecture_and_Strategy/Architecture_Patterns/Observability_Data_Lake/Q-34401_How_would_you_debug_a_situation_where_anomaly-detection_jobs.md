---
type: "practice"
difficulty: "Deep"
---

# How would you debug a situation where anomaly-detection jobs reading from the lake suddenly slowed 5x after a schema evolution event?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Detecting Ingestion Changes Behind Drift](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Detecting_Ingestion_Changes_Behind_Drift.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When debugging an issue like a sudden slowdown in anomaly-detection jobs after a schema evolution event, we need to systematically understand what changed and how it affected data processing. Here’s the step-by-step approach from first principles:

Firstly, this situation involves observing the interactions between the AIOps infrastructure (the data lake and its associated services) and the anomaly detection process. The key steps are to identify potential bottlenecks, analyze performance metrics, and validate the schema evolution impact.

The question essentially probes understanding of observability practices in a complex system, performance tuning, and debugging techniques for distributed systems. We assume that the AIOps data lake stores large volumes of time-series data, and the anomaly detection jobs rely on this data for real-time or batch analysis. The schema evolution likely involved adding new columns, changing data types, or altering indexes.

Let's break down the solution step-by-step:

Identify the Bottleneck: Start by checking if the slowdown is due to resource constraints like CPU, memory, network I/O, or disk I/O.
Review Recent Changes: Examine the schema evolution event and understand how it impacted data storage and access patterns.
Analyze Metrics: Look at relevant metrics such as job execution time, latency, throughput, and error rates to pinpoint where the slowdown occurs.
Validate Data Intake and Processing: Ensure that the new schema is correctly applied and data ingestion is not being bottlenecked.

Let's dive into each step in detail:

Step 1: Identify the Bottleneck

Use observability tools such as Prometheus or Datadog to monitor the following metrics:

CPU usage
Memory usage
Network I/O (e.g., network requests per second)
Disk I/O (read/write operations)

For example, you can run a command like:

BASH
top -b -n 1 | grep [p]rocess-name > process_cpu_usage.txt

Or use Prometheus queries to inspect metrics:

PROMQL
rate(process_cpu_seconds_total[5m])

This will help identify if any specific service is under heavy load.

Step 2: Review Recent Changes

Review the schema evolution event. Consider these aspects:

What columns were added or modified?
Were there any index changes that might impact query performance?

For example, if a new column was added as a string type and not indexed, it could significantly slow down queries involving this column.

Step 3: Analyze Metrics

Check the job execution time and latency:

PROMQL
avg by (job) (histogram_quantile(0.95, sum(rate(anomaly_detection_job_duration_seconds_bucket[1m])) by (le)))

This will give you an idea of where 95% of the jobs are spending most of their time.

Step 4: Validate Data Intake and Processing

Ensure that data ingestion is not being bottlenecked:

Check if there are any queueing delays or backlogs.
Verify that the new schema is correctly applied during ingestion.

For example, you can monitor the number of failed ingestions:

PROMQL
anomaly_detection_job_error_count_total

#### Closing Summary
and Common Mistakes

By systematically analyzing these aspects, we can pinpoint the root cause of the slowdown. Common mistakes include ignoring distributed system complexities, not validating schema evolution impacts, or misinterpreting performance metrics.

To ensure completeness:

Verify that all relevant services are monitored.
Ensure that data ingestion is smooth post-schem change.
Validate that job execution patterns have changed as expected.

