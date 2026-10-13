---
type: "practice"
difficulty: "Intermediate"
---

# How would you compute the count of log entries per service per severity in one query using conditional aggregation?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Conditional Aggregation](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Conditional_Aggregation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute the count of log entries per service per severity, we can use SQL's GROUP BY clause along with conditional aggregation. This approach allows us to categorize logs by service and severity while counting them efficiently in a single query.

#### Triaging the Question

The question is asking for a way to group log entries by both "service" and "severity," then count these groups. The key here is leveraging SQL's COUNT function with a CASE statement inside, which enables us to conditionally aggregate data based on criteria.

#### First-Principles Solution

Let's assume we have a logs table with the following schema:

SQL
CREATE TABLE logs (
    id INT,
    service VARCHAR(255),
    severity VARCHAR(10)
);

To compute the count of log entries per service and per severity, we can use the following SQL query:

SQL
SELECT
    service,
    severity,
    COUNT(CASE WHEN severity = 'ERROR' THEN 1 ELSE NULL END) AS error_count,
    COUNT(CASE WHEN severity = 'WARNING' THEN 1 ELSE NULL END) AS warning_count,
    COUNT(CASE WHEN severity = 'INFO' THEN 1 ELSE NULL END) AS info_count
FROM
    logs
GROUP BY
    service, severity;

#### Closing Summary
and Common Mistakes

This query uses COUNT with CASE statements to conditionally count entries based on the severity. By grouping by both service and severity, we get a breakdown of log counts for each combination.

#### Common Mistakes
Forgetting to group by all necessary columns.
Using COUNT(*) without conditions, which would just give the total number of rows rather than conditionally counting.
Not using ELSE NULL END in the CASE statement, leading to incorrect results when there are no matching entries.

#### Example
Data

For a more concrete example, let's say we have the following data:

SQL
INSERT INTO logs (id, service, severity) VALUES
(1, 'web', 'ERROR'),
(2, 'api', 'WARNING'),
(3, 'web', 'INFO'),
(4, 'api', 'ERROR'),
(5, 'web', 'WARNING');
Worked Input-Output

Running the query on this data will yield:

SQL
service | severity  | error_count | warning_count | info_count
--------|-----------|-------------|---------------|------------
web     | ERROR     | 1           | 0             | 1
web     | WARNING   | 0           | 1             | 0
api     | ERROR     | 1           | 0             | 0
api     | WARNING   | 0           | 1             | 0
Summary

In summary, we used SQL's GROUP BY and conditional aggregation to count log entries per service per severity. The key is ensuring that all necessary columns are grouped correctly and using CASE statements inside the COUNT function for accurate condition-based counting.

**Comparison:**

| service | severity  | error_count | warning_count | info_count |
|---------|-----------|-------------|---------------|------------|
| web     | ERROR     | 1           | 0             | 1          |
| web     | WARNING   | 0           | 1             | 0          |
| api     | ERROR     | 1           | 0             | 0          |
| api     | WARNING   | 0           | 1             | 0          |

**Code:**

```sql
SELECT
    service,
    severity,
    COUNT(CASE WHEN severity = 'ERROR' THEN 1 ELSE NULL END) AS error_count,
    COUNT(CASE WHEN severity = 'WARNING' THEN 1 ELSE NULL END) AS warning_count,
    COUNT(CASE WHEN severity = 'INFO' THEN 1 ELSE NULL END) AS info_count
FROM
    logs
GROUP BY
    service, severity;
```

**Example input / output:**

```text
SELECT
    service,
    severity,
    COUNT(CASE WHEN severity = 'ERROR' THEN 1 ELSE NULL END) AS error_count,
    COUNT(CASE WHEN severity = 'WARNING' THEN 1 ELSE NULL END) AS warning_count,
    COUNT(CASE WHEN severity = 'INFO' THEN 1 ELSE NULL END) AS info_count
FROM
    logs
GROUP BY
    service, severity;

-- Output:
service | severity  | error_count | warning_count | info_count
--------|-----------|-------------|---------------|------------
web     | ERROR     | 1           | 0             | 1
web     | WARNING   | 0           | 1             | 0
api     | ERROR     | 1           | 0             | 0
api     | WARNING   | 0           | 1             | 0
```

**Data:**

```sql
CREATE TABLE logs (
    id INT,
    service VARCHAR(255),
    severity VARCHAR(10)
);

INSERT INTO logs (id, service, severity) VALUES
(1, 'web', 'ERROR'),
(2, 'api', 'WARNING'),
(3, 'web', 'INFO'),
(4, 'api', 'ERROR'),
(5, 'web', 'WARNING');
```

**Flow:**

1. Yes
2. No
3. Start
4. Are all necessary columnsgrouped?
5. Run the query
6. Observe results
7. Add missing columns toGROUP BY clause
8. End

