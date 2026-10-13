---
type: "practice"
difficulty: "Intermediate"
---

# Your OLAP dashboard shows yesterday's data missing until noon — walk through how ETL SLAs, dependency DAGs, and freshness monitoring would catch that earlier.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Separating OLTP and Analytics](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to understand why yesterday's data is only partially loaded by noon in an OLAP dashboard. The core concepts involve ETL (Extract, Transform, Load) service level agreements (SLAs), dependency Directed Acyclic Graphs (DAGs), and freshness monitoring mechanisms. Let’s break down each component step-by-step.

Firstly, ETL SLAs ensure that data is loaded within a specified timeframe to meet business requirements. In this case, the missing yesterday's data until noon suggests there might be an issue with meeting the agreed-upon ETL window.

Secondly, dependency DAGs manage the order and interdependence of various ETL processes. If one process fails or lags, it can block subsequent tasks from starting, which could explain why only part of yesterday's data is missing.

Lastly, freshness monitoring helps detect when data has not been loaded within acceptable timeframes. By setting up a robust freshness monitor, we can pinpoint the exact point in the ETL process where things went wrong.

Let’s dive into each component:

ETL SLA:

Problem Identification: The ETL job should complete by noon to ensure yesterday's data is available for the dashboard.
Root Cause Analysis: Check if the ETL job was designed with sufficient parallelism and resource allocation. Ensure that any long-running processes or bottlenecks are identified and mitigated. For instance, if a single large table update was taking too long, it might delay the entire job.

Dependency DAG:

Graph Structure: The dependency graph should clearly show which tasks depend on others. If there’s an issue with a dependent task, e.g., a data transformation or loading step, this could block downstream processes.

#### Example Scenario
Suppose a critical ETL process that loads intermediate data into a staging area is delayed due to high latency in source systems. This would affect the next steps (e.g., aggregations) and prevent them from starting until the delay is resolved.

Freshness Monitoring:

Monitoring Mechanisms: Implement a system that continuously checks the freshness of ETL outputs. For example, set up alerts if data older than 12 hours is detected.
Alerts & Notifications: Configure notifications for stakeholders to act quickly when issues arise. This could involve Slack or email alerts to team members.

Now, let’s reason through a possible solution from first principles:

Initial Checks:

Verify the ETL job logs to see if any steps failed or took longer than expected.
Review the dependency DAG for any delays in critical tasks.

Detailed Analysis:

If a particular task is consistently taking longer, investigate its performance. This could involve optimizing queries, increasing resources, or rearchitecting parts of the ETL process.
For instance, if a transformation step was causing bottlenecks, consider breaking it into smaller chunks that can run concurrently.

Implement Freshness Monitoring:

Set up a script to periodically check the freshness of ETL outputs. This could be as simple as querying the last updated timestamp in a database table.
Use CI/CD tools like Jenkins or CircleCI to automate this process and send alerts if data is not fresh.

Continuous Improvement:

Regularly review and update SLAs based on actual performance metrics.
Implement feedback loops where team members can suggest improvements to the ETL process.

In conclusion, the combination of robust ETL SLAs, well-defined dependency DAGs, and proactive freshness monitoring will help catch issues like missing data earlier. This approach ensures timely and accurate data delivery for business-critical dashboards.

#### Common mistakes
include not having a clear dependency graph, failing to monitor data freshness effectively, or simply not allocating enough resources to critical tasks.

