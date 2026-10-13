---
type: "practice"
difficulty: "Intermediate"
---

# Describe how you would build a top-N slow queries report that engineers actually use and act on.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Percentiles vs Averages](<../../../../01_Databases_and_SQL/Database_Operations/Monitoring/Percentiles_vs_Averages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build an effective top-N slow queries report, we need to capture detailed performance metrics of database queries and present them in a format that is actionable by the engineering team. The goal is to identify and address slow-performing queries to improve overall system performance.

First, let's clarify what "actually use and act on" means: it implies that the report should provide sufficient detail for engineers to diagnose and optimize problematic queries without needing additional information or context.

#### Triaging the Question
What the question is asking: How to build a slow queries report that engineers can use to identify and address performance issues.

#### Assumptions
The database system in use supports query profiling, and we have access to logging mechanisms to capture necessary data. We are using a relational database like PostgreSQL or MySQL for this example.

#### First Principles Solution

Collect Query Performance Data:

Monitor slow queries by setting thresholds for execution time.
Collect metadata such as the query text, start time, end time, duration, and resource usage (CPU, memory).

Store the Data Efficiently:

Design a schema that allows efficient querying of the collected data.
Use database-specific tools or system catalogs to gather performance metrics.

Aggregate and Rank Queries:

Aggregate the data by query text to identify recurring slow queries.
Rank these queries based on total execution time, average execution time, or other relevant criteria.

Present the Report:

Generate a report that includes detailed information about each query.
Provide visualizations (e.g., charts) to highlight trends and outliers.

Ensure Usability:

Make the report easily accessible through a web interface or dashboard.
Include context such as database schema, execution plan, and any relevant application logs.
Detailed Steps

Set Up Query Profiling:

SQL
-- Enable slow query logging in PostgreSQL
SET statement_timeout = 5000; -- Set timeout to 5 seconds
EXPLAIN ANALYZE SELECT * FROM large_table;

Design the Schema:

Create a table to store query performance data.
SQL
CREATE TABLE slow_queries (
    id SERIAL PRIMARY KEY,
    query TEXT NOT NULL,
    start_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    end_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    duration DOUBLE PRECISION,
    cpu_used DOUBLE PRECISION,
    memory_used DOUBLE PRECISION,
    user_id INT REFERENCES users(id)
);

Insert Performance Data:

Trigger an event to log slow queries automatically.
SQL
CREATE OR REPLACE FUNCTION log_slow_query()
RETURNS TRIGGER AS $$
BEGIN
    IF (EXTRACT(EPOCH FROM AGE(TG_ARGV[0], clock_timestamp())) * 1000) > TG_ARGV[1] THEN
        INSERT INTO slow_queries (query, start_time, end_time, duration)
        VALUES (TG_ARGV[2], clock_timestamp(), clock_timestamp() + (AGE(TG_ARGV[0], clock_timestamp())), EXTRACT(EPOCH FROM AGE(TG_ARGV[0], clock_timestamp())));
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER log_slow_queries_trigger
AFTER STATEMENT ON public.schemas_table
FOR EACH STATEMENT
EXECUTE FUNCTION log_slow_query('5 seconds', 'SELECT * FROM large_table');

Aggregate and Rank Queries:

SQL
WITH query_stats AS (
    SELECT
        query,
        COUNT(*) AS executions,
        SUM(duration) AS total_duration,
        AVG(duration) AS avg_duration,
        MAX(cpu_used) AS max_cpu_used,
        MAX(memory_used) AS max_memory_used
    FROM slow_queries
    GROUP BY query
)
SELECT * FROM query_stats ORDER BY total_duration DESC LIMIT 10;

Generate the Report:

Use a web framework (e.g., Django, Flask) to create a dashboard.
PYTHON
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/slow_queries')
def slow_queries():
    queries = query_stats()  # Assume this function fetches the top-N slow queries
    return render_template('queries.html', queries=queries)

if __name__ == '__main__':
    app.run(debug=True)

#### Closing Summary
and Common Mistakes/Follow-Ups
Ensure the report is regularly updated to reflect current performance issues.

#### Cons
ider including context such as database schema changes or recent application deployments that might affect query performance.
Address potential limitations like false positives in slow query detection (e.g., long-running but non-blocking queries).

#### Follow-ups
could involve discussing more advanced features like query optimization suggestions, integration with CI/CD pipelines for proactive issue resolution.

**Comparison:**

```
Column	Type	Description
id	SERIAL	Unique identifier for each record
query	TEXT	The SQL query text
start_time	TIMESTAMP	Timestamp when the query started
end_time	TIMESTAMP	Timestamp when the query ended
duration	DOUBLE PRECISION	Time taken to execute the query
cpu_used	DOUBLE PRECISION	CPU usage during execution
memory_used	DOUBLE PRECISION	Memory usage during execution
user_id	INT	ID of the user who executed the query
```

**Flow:**

1. Yes
2. Start
3. Is slow query logging enabled?
4. Monitor and log slow queries
5. Aggregate data by query text
6. Rank queries based on totalduration
7. Presentation layer (dashboard)
8. End

