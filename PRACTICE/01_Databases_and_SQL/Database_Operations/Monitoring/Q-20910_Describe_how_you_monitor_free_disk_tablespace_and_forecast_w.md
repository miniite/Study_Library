---
type: "practice"
difficulty: "Intermediate"
---

# Describe how you monitor free disk/tablespace and forecast when it will run out.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Database Health Signals](<../../../../01_Databases_and_SQL/Database_Operations/Monitoring/Database_Health_Signals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To monitor free disk/tablespace in a database environment, we need to track available storage resources and predict potential future shortages. This is crucial for maintaining system performance and preventing critical failures. Here’s how you would approach this task step by step.

Firstly, the question asks us to describe how to monitor free disk/tablespace and forecast when it will run out. The core idea is to continuously check the available storage space, set up alerts for thresholds, and perform predictive analysis using historical data and trends.

To break down the solution from first principles:

Collect Current Disk/Tablespace Usage Data: Use database-specific commands or tools like df -h (for file systems) or SELECT * FROM dba_datafile; (for Oracle databases) to gather current disk usage details.
Set Up Monitoring Alerts: Configure alerts for critical thresholds, such as when the free space falls below 10% of total storage.
Analyze Historical Data: Use time-series data analysis techniques to forecast future trends in storage use.
Implement Proactive Management Strategies: Based on predictions, implement strategies like archiving older data or resizing volumes.

Now, let's go through each step in detail:

#### Triaging the Question

The interviewer is probing your understanding of database monitoring tools, alert systems, and predictive analytics. They are also interested in how you handle trade-offs between resource usage thresholds and performance impacts.

#### First-Principles Solution
Step 1: Collect Current Disk/Tablespace Usage Data

For a PostgreSQL database, the pg_stat_database system catalog can be queried to get current disk usage:

SQL
SELECT datname, datfreespace/1024/1024 AS free_space_mb FROM pg_stat_database;

This gives you the free space in MB for each database.

For file systems, use df -h or a similar command-line tool to get overall disk usage:

SH
df -h
Step 2: Set Up Monitoring Alerts

Use a monitoring system like Prometheus, Grafana, or custom scripts with tools like Nagios. For example, using a script in Bash to check and alert:

BASH
#!/bin/bash

FREE_SPACE_THRESHOLD=10 # in GB

df -h | grep '/$' | awk '{print $4}' | while read line; do
    free_space=$(echo "$line" | cut -d '%' -f 1)
    if [ "$(echo "$free_space < $FREE_SPACE_THRESHOLD" | bc)" -eq 1 ]; then
        echo "Disk space is critically low: $free_space%"
        # Send alert via email or Slack
    fi
done
Step 3: Analyze Historical Data

Use time-series databases like TimescaleDB to store and analyze historical storage usage data. Perform queries like:

SQL
SELECT datname, date_trunc('day', now()) AS day, AVG(datfreespace/1024/1024) as avg_free_space_mb
FROM pg_stat_database
GROUP BY datname, day;

Plot this data using tools like Grafana to identify trends and patterns.

Step 4: Implement Proactive Management Strategies

Based on predictive analysis, implement strategies such as:

Archiving: Move older data to secondary storage.
Storage Optimization: Optimize database queries or indexes that consume significant space.
Volume Resizing: Increase disk volume capacity if needed.

#### Closing Summary
and Common Mistakes/Follow-Ups

Continuous monitoring is essential to prevent outages due to lack of disk space. A robust solution includes not just alerting, but also proactive management strategies. Common mistakes include setting very low thresholds (too many false positives) or ignoring the historical data trends. Follow-ups could involve more details on specific tools used for monitoring and how to integrate these solutions into existing workflows.

