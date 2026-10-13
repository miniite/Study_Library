---
type: "practice"
difficulty: "Basic"
---

# Why might a LEFT JOIN be safer than an INNER JOIN when joining an alerts table to a hosts table for a dashboard?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Join Types](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When building a dashboard that requires information from both an alerts table and a hosts table, using a LEFT JOIN can be safer than an INNER JOIN because it ensures all records from the hosts table are included in the result set, even if there is no corresponding record in the alerts table. This approach helps maintain data integrity and provides a more comprehensive view of your host data.

#### Triage

The question is probing why a LEFT JOIN might be preferred over an INNER JOIN when joining these two tables for a dashboard. We need to understand that an INNER JOIN returns only those records where there is a match in both tables, whereas a LEFT JOIN includes all records from the left table (in this case, hosts) and any matching records from the right table (alerts), which might be useful when visualizing data.

#### Assumptions
Both hosts and alerts are relational databases with a common key such as host_id.
The goal is to display all hosts on the dashboard, whether or not they have corresponding alerts.
The presence of an alert (or lack thereof) should be represented in the results.

#### First-Principles Solution

Let's consider the following scenario:

You are building a dashboard that tracks server health. The hosts table contains information about all servers, and the alerts table logs any issues or incidents on these hosts.
If you use an INNER JOIN, only hosts with corresponding alerts will be included in the result set. This might leave out important hosts without current issues, which could skew your dashboard's representation of server health.

Here’s how a LEFT JOIN would work:

SQL
SELECT *
FROM hosts h
LEFT JOIN alerts a ON h.host_id = a.host_id;

This query ensures that all hosts are included in the result set. If there is no alert for a particular host, the columns from the alerts table will contain NULL values.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, using a LEFT JOIN keeps your dashboard comprehensive by including all hosts, even if they don't have any current alerts. This can be safer because it prevents you from missing important data points that might affect the overall health status of servers.

#### Common mistakes
include:

Overlooking the need to include non-matching records.
Assuming an INNER JOIN is sufficient for all join scenarios.
Ignoring the possibility of NULL values in columns from the right table during a LEFT JOIN, which can be handled with appropriate filtering or data processing steps.

#### Example
Query
SQL
SELECT h.host_name, a.alert_id, a.message
FROM hosts h
LEFT JOIN alerts a ON h.host_id = a.host_id;

This query will return all host names along with any related alert information. If a host has no associated alert, the alert_id and message columns will be NULL.

Self-Check

Correctness + completeness confirmed.

**Code:**

```sql
SELECT h.host_name, a.alert_id, a.message
FROM hosts h
LEFT JOIN alerts a ON h.host_id = a.host_id;
```

