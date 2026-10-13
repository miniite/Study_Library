---
type: "practice"
difficulty: "Deep"
---

# What is the difference between a scheduled full refresh and a CDC-based incremental pipeline, and when would a fresher pick each?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Incremental Rollups](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Incremental_Rollups.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

A scheduled full refresh involves completely reprocessing all historical data at regular intervals, while a CDC (Change Data Capture)-based incremental pipeline processes only the changes made to the data since the last update. A fresher would pick a CDC-based approach for its efficiency and scalability, but might opt for a full refresh in scenarios where historical completeness is more critical.

#### Triaging the Question

The core of this question revolves around understanding the trade-offs between two different methods of maintaining up-to-date datasets: scheduled full refreshes and CDC-based incremental pipelines. The interviewer is probing your grasp of fundamental data engineering concepts, your ability to analyze use cases, and your decision-making skills in choosing appropriate strategies.

#### Assumptions
We are discussing real-time or near-real-time processing.
The dataset has a high volume and frequent changes.
The goal is to maintain an up-to-date dataset with minimal overhead.

#### First Principles Solution
1. Scheduled Full Refresh:

A scheduled full refresh involves running ETL (Extract, Transform, Load) jobs periodically to reprocess the entire dataset from scratch. This approach ensures that all historical data is completely processed and up-to-date at each run. It's straightforward but can be resource-intensive and slow for large datasets.

#### Pros

Ensures complete and consistent data.
Simplifies error handling; if a job fails, it can start afresh.

#### Cons

High computational cost, especially with large historical data sets.
Long processing times during each refresh.
Potentially high resource usage (CPU, memory) for the entire dataset.
2. CDC-Based Incremental Pipeline:

A CDC-based incremental pipeline processes only the changes (inserts, updates, deletes) made to the source dataset since the last update. This approach is designed to be more efficient and scalable by processing only what has changed.

#### Pros

Highly efficient for datasets with frequent updates.
Low resource usage because it doesn't reprocess entire data sets.
Can maintain real-time or near-real-time data.
Allows for easier scaling as the size of the dataset grows.

#### Cons

Requires a more complex setup to capture and apply changes (e.g., using triggers, logs).
May require handling edge cases like transactional consistency.
Slightly more challenging error handling; partial success of an incremental job might leave the data inconsistent.
When Would You Pick Each?
Full Refresh:
Historical Completeness is Critical: If your use case requires a completely up-to-date dataset at every refresh, regardless of computational cost or resource usage.
Low Volume Data Sets: For smaller datasets where the overhead of a full refresh doesn't significantly impact performance.
Periodic, Non-Real-Time Scenarios: When real-time processing is not critical and periodic updates are sufficient.
Incremental CDC:
High Volume Data: When dealing with large data sets that experience frequent changes.
Real-Time or Near-Real-Time Requirements: For scenarios where the dataset needs to be updated as changes occur.
Cost Efficiency: In environments where resource usage and computational costs are a concern.

#### Closing Summary

For a fresher, understanding both methods is crucial. CDC-based pipelines offer greater efficiency and scalability, making them ideal for most modern data engineering projects. However, in certain scenarios like historical completeness requirements or low-volume datasets, scheduled full refreshes might be the better choice.

#### Common Mistakes/Follow-Ups
Failing to Consider Edge Cases: For example, handling partial failures in CDC pipelines.
Not Understanding the Impact on Data Consistency: Ensuring that incremental updates don’t leave the data in a partially updated state.
Overlooking Resource Management: Full refreshes can be resource-intensive and should be carefully managed.

Would you like to explore any of these aspects further, or are there specific use cases you want to discuss?

**Comparison:**

```
change_id	order_id	action	new_product	new_quantity
3	1	UPDATE	Laptop	6

--- Final incremental update query

SQL
Copy
SELECT * FROM OrderChanges WHERE timestamp > '2023-10-01 00:00:00';
change_id	order_id	action	new_product	new_quantity
3	1	UPDATE	Laptop	6
```

**Code:**

```

-- Example CDC-based pipeline logic

CREATE TABLE Orders ( id INT, product VARCHAR(255), quantity INT, updated_at TIMESTAMP );

CREATE TABLE OrderChanges ( change_id INT AUTO_INCREMENT PRIMARY KEY, order_id INT, action CHAR(6), -- 'INSERT', 'UPDATE', 'DELETE' new_product VARCHAR(255), new_quantity INT, timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP );

-- Inserting initial data for demonstration

INSERT INTO Orders (id, product, quantity, updated_at) VALUES (1, 'Laptop', 10, NOW());

CODE
Copy

```sql
-- CDC logic to capture changes

DELIMITER //
CREATE TRIGGER orders_insert AFTER INSERT ON Orders FOR EACH ROW
BEGIN
    INSERT INTO OrderChanges (order_id, action, new_product, new_quantity)
    VALUES (NEW.id, 'INSERT', NEW.product, NEW.quantity);
END;//
DELIMITER ;
SQL
Copy
-- Query to fetch incremental updates

SELECT * FROM OrderChanges WHERE timestamp > '2023-10-01 00:00:00';
```

**Example input / output:**

```

-- Initial input data for Orders table

INSERT INTO Orders (id, product, quantity) VALUES (1, 'Laptop', 10), (2, 'Monitor', 5);

-- CDC trigger creates these records in OrderChanges after each insert into Orders

SELECT * FROM OrderChanges;

CODE
Copy
| change_id | order_id | action  | new_product  | new_quantity |
|-----------|---------|--------|-------------|--------------|
|         1 |       1 | INSERT | Laptop      |           10 |
|         2 |       2 | INSERT | Monitor     |            5 |

-- Incremental update for the Orders table

UPDATE Orders SET quantity = 6 WHERE id = 1;

-- CDC trigger creates this record in OrderChanges after each update to Orders
```sql
INSERT INTO OrderChanges (order_id, action, new_product, new_quantity)
VALUES (1, 'UPDATE', NULL, 6);
```

**Data:**

```json
{
    "orders": [
        { "id": 1, "product": "Laptop", "quantity": 10 },
        { "id": 2, "product": "Monitor", "quantity": 5 }
    ],
    "order_changes": [
        { "change_id": 1, "order_id": 1, "action": "INSERT", "new_product": "Laptop", "new_quantity": 10 },
        { "change_id": 2, "order_id": 2, "action": "INSERT", "new_product": "Monitor", "new_quantity": 5 }
    ]
}
```

**Flow:**

1. Yes
2. No
3. Initial Data Load
4. Is it a Full Refresh?
5. Process Entire Dataset
6. CDC-based IncrementalPipeline
7. Process Changes Only
8. Handle Errors
9. Capture and Apply Changes
10. Ensure Data Consistency

