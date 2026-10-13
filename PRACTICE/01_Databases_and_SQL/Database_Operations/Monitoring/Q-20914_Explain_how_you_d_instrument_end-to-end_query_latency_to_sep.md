---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd instrument end-to-end query latency to separate network time from actual DB execution time.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Percentiles vs Averages](<../../../../01_Databases_and_SQL/Database_Operations/Monitoring/Percentiles_vs_Averages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To instrument end-to-end query latency and accurately separate network time from database execution time, we need a robust monitoring strategy that captures both network and execution metrics. This involves designing a system that can measure the full lifecycle of a query, including its journey through the network, and then pinpointing where the actual database processing begins and ends.

#### Triage

The question is asking for a method to distinguish between network latency and database execution time in end-to-end query latency monitoring. The key aspects are understanding how to accurately capture these components and how to aggregate them properly.

#### First-Principles Solution

Capture Network Time:

Instrument the client-side to measure when the query is sent to the server.
Measure the response time on the client side, which includes both network latency and initial database processing.

Capture Database Execution Time:

Implement a logging mechanism within the database that records the start and end times of query execution at different stages (e.g., parsing, planning, actual execution).
Use low-overhead methods to minimize performance impact on the database.

Aggregate Metrics:

On the client side, subtract the network time from the total response time to get the database processing time.
Store and analyze these metrics for comprehensive monitoring.
Step-by-Step Solution

Client-Side Instrumentation:

PYTHON
import time

def send_query(query):
    start_time = time.time()
    # Send query to server
    response_time = time.time() - start_time
    return response_time, query

Database Side Logging:

SQL
-- Enable detailed logging
SET enable_log_statement = 'true';

-- Example of a log statement in SQL (PostgreSQL)
EXPLAIN ANALYZE SELECT * FROM table WHERE condition;
Use EXPLAIN ANALYZE to capture detailed execution times.
Log the start and end timestamps for each query at different stages.

Aggregation Logic:

PYTHON
def calculate_metrics(client_response_time, db_logs):
    network_time = client_response_time - (db_logs['parse_time'] + db_logs['plan_time'])
    database_processing_time = db_logs['total_execution_time']

    return network_time, database_processing_time

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Failing to account for network latency in the client-side measurement.
Not properly logging execution times within the database, leading to incorrect separation of metrics.

#### Follow-Up
Questions:

How would you handle different types of queries (e.g., read-heavy vs write-heavy)?
What about cross-datacenter scenarios where additional network time is involved?
How do you ensure minimal performance impact on the database during logging?
Depth and Language Match

Given the role of a Database Developer, this solution needs to be precise yet approachable. The explanation covers fundamental concepts while providing actionable steps that can be implemented in a real-world scenario.

**Code:**

```

import time

def send_query(query): start_time = time.time() # Simulate sending query to server response_time = time.time() - start_time return response_time, query

CODE
Copy

```sql
-- Example of a log statement in SQL (PostgreSQL)
EXPLAIN ANALYZE SELECT * FROM table WHERE condition;
```

