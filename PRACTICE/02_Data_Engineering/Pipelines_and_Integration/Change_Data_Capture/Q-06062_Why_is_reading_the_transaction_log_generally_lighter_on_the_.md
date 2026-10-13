---
type: "practice"
difficulty: "Deep"
---

# Why is reading the transaction log generally lighter on the source than running trigger-based CDC?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ AWS DMS vs Debezium](<../../../../02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/AWS_DMS_vs_Debezium.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Reading the transaction logs directly from the database is often more efficient and lightweight compared to implementing a trigger-based change data capture (CDC) system. This approach minimizes overhead on the source system, as it does not require additional processing or storage resources that triggers would consume.

#### Triaging the Question

The question essentially asks why reading transaction logs directly can be less resource-intensive for the source database than using triggers to capture changes. The core of this problem involves understanding how different CDC methods interact with the source database and their respective performance implications.

Key Assumptions:
Transaction Logs: These are system-generated records that track all data modifications, including inserts, updates, and deletes.
Triggers: Database objects that execute automatically in response to certain events (e.g., insertions, updates).

#### First Principles Solution

To understand the difference between these two methods, we need to consider how they interact with the database and the overhead they introduce.

Transaction Logs:

Direct Reading:

The source database maintains a transaction log that records all changes made to the data.
A CDC system can read this log directly to identify changes without needing any additional infrastructure on the source side.
This method is lightweight because it only requires reading existing logs, which are already maintained by the database.

Resource Impact:

Direct access to transaction logs typically doesn't add significant overhead since these logs are designed for auditing and recovery purposes.
The CDC system can use database APIs or specific log parsers provided by the database vendor to read these logs efficiently.
Triggers:

#### Implementation

Triggers must be explicitly created on the source tables, which introduces additional metadata and complexity.
These triggers are part of the table structure and need to be maintained separately from regular data management operations.

Resource Impact:

Triggers can introduce performance overhead because they run for every insert, update, or delete operation on the relevant tables.
Additional storage is required to manage these triggers, which adds to the overall database footprint.
The frequent firing of triggers can lead to increased CPU and I/O usage, impacting the source system's performance.

#### Trade-offs
Transaction Logs: Low overhead but requires a robust CDC mechanism to parse logs correctly.
Triggers: Higher initial setup cost in terms of development and maintenance but potentially easier integration with existing systems.

#### Closing Summary

In summary, reading transaction logs directly is generally more lightweight because it leverages the database's built-in mechanisms without adding additional overhead. Triggers, while providing a simpler implementation path, can introduce significant performance and resource costs on the source system.

#### Common Mistakes/Follow-Ups
Assuming Triggers Always Increase Overhead: While triggers do add some overhead, modern databases handle them efficiently in most cases.
Ignoring Database-Specific Capabilities: Some databases offer built-in CDC features that can be as efficient as direct log reading without the need for custom triggers.
Overlooking Performance Monitoring: Both methods should be monitored to ensure they meet performance requirements.

**Comparison:**

```
Table	Column	Type
orders	order_id	INT
	product	VARCHAR(50)
	quantity	INT
Products	Column	Type
prod_id	INT
	name	VARCHAR(100)
	price	DECIMAL(10,2)
```

**Code:**

```python
# Example code snippet to read transaction logs (hypothetical)
def read_transaction_logs(database_connection):
    # Code to query and process transaction logs
    pass

# Example trigger implementation (hypothetical)
trigger_code = """
CREATE TRIGGER update_log
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    -- Log the changes
END;
"""
```

**Example input / output:**

```text
Input:
- Transaction logs from MySQL database

Output:
- List of changes (inserts, updates, deletes) for specific tables
```

**Data:**

```json
{
  "database": {
    "name": "exampleDB",
    "type": "MySQL"
  },
  "transaction_logs": [
    {"id": 1, "action": "INSERT", "table": "orders", "time": "2023-10-01T09:00:00Z"},
    {"id": 2, "action": "UPDATE", "table": "products", "time": "2023-10-01T09:30:00Z"}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Read Transaction Logs
4. Is the database configured forCDC?
5. Implement Triggers
6. Directly Access Log Data
7. Monitor Performance
8. Optimize as Needed

