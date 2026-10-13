---
type: "practice"
difficulty: "Deep"
---

# Explain how DLT handles CDC with APPLY CHANGES INTO and what it does under the hood versus a hand-written MERGE.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Delta Live Tables and Serving](<../../../../TOOLS/Databricks/Databricks_Platform/Delta_Live_Tables_and_Serving.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

DLT (Delta Live Tables) is a powerful feature in Databricks that simplifies handling Change Data Capture (CDC) with its APPLY CHANGES INTO command. At a high level, DLT processes CDC data efficiently and ensures data integrity by leveraging Delta Lake's transactional capabilities. This answer will explain how DLT handles CDC using APPLY CHANGES INTO, contrast it with a hand-written MERGE operation, and delve into the underlying mechanisms.

Firstly, let’s clarify what we are comparing:

DLT with APPLY CHANGES INTO: A built-in feature of Databricks that provides an elegant way to process CDC data.
Hand-written MERGE: A custom SQL solution where you manually write a MERGE statement to handle upserts (insert or update).

Under the hood, both methods aim to perform upsert operations—inserting new records and updating existing ones. However, DLT abstracts away much of the complexity involved in managing transactions and ensuring data consistency.

#### Triage

The question is probing your understanding of:

Delta Lake's capabilities for handling CDC.
DLT’s APPLY CHANGES INTO syntax and its benefits over a hand-written MERGE.
Transaction management in both approaches.

#### Performance considerations
when dealing with large volumes of data.

#### First-Principles Solution
Delta Lake's Capabilities

Delta Lake provides a durable storage layer for big data applications, ensuring ACID (Atomicity, Consistency, Isolation, Durability) transactions. DLT leverages these capabilities to handle CDC efficiently.

APPLY CHANGES INTO

APPLY CHANGES INTO is a feature in Databricks that simplifies the process of applying CDC data into an existing Delta table. Here’s how it works:

Stream Processing: It processes streaming data in real-time, which is essential for CDC.
Delta Transactions: It uses Delta Lake's transactional features to ensure atomicity and consistency during upsert operations.
Hand-Written MERGE

A hand-written MERGE operation involves writing a custom SQL statement that checks for the existence of records (using conditions) and performs inserts or updates accordingly. This approach requires careful handling of transactions to avoid concurrency issues, data loss, or integrity violations.

Step-by-Step Solution

DLT with APPLY CHANGES INTO:

Input Stream: Data is ingested into a DLT pipeline from a source like Apache Kafka.
Transformation Layer: The data goes through transformations (e.g., filtering, enriching).
APPLY CHANGES INTO: The final step applies the CDC changes to an existing Delta table. This operation ensures that all transactions are handled atomically and consistently.

Hand-Written MERGE:

Input Stream: Data is ingested into a temporary location.
Transformation Layer: Data undergoes transformations similar to DLT.
MERGE Operation: Custom SQL MERGE statements are used to upsert data. This requires careful management of transactions and concurrency.

#### Closing Summary

In summary, DLT’s APPLY CHANGES INTO abstracts away much of the complexity involved in managing CDC data by leveraging Delta Lake’s robust transactional features. It ensures consistency, atomicity, and performance without requiring you to manually handle these aspects. In contrast, a hand-written MERGE operation provides more control but requires careful handling of transactions and concurrency.

#### Common Mistakes
and Follow-Ups
Common Mistake: Failing to understand the transactional benefits provided by Delta Lake.

#### Follow-Up
Exploring scenarios where hand-written MERGE might be preferable, such as custom business logic that is not easily handled by DLT’s built-in features.

#### Example
Sample Data Schema and JSON
JSON
{
  "schema": {
    "fields": [
      {"name": "id", "type": "string"},
      {"name": "value", "type": "int"}
    ]
  }
}

#### Example
WORKED INPUT->OUTPUT / DRY-RUN
SQL
-- Input data stream
INSERT INTO delta_table VALUES (1, 20)
INSERT INTO delta_table VALUES (2, 30)

-- Apply changes into existing Delta table
APPLY CHANGES INTO delta_table FROM changes_stream

-- Hand-written MERGE operation
MERGE INTO delta_table USING changes_stream ON delta_table.id = changes_stream.id WHEN MATCHED THEN UPDATE SET value = changes_stream.value WHEN NOT MATCHED THEN INSERT (id, value) VALUES (changes_stream.id, changes_stream.value)

**Comparison:**

| id | value |
|----|-------|
| 1  | 20    |
| 2  | 30    |

**Code:**

```sql
-- DLT APPLY CHANGES INTO Example
APPLY CHANGES INTO delta_table FROM changes_stream

-- Hand-written MERGE Example
MERGE INTO delta_table USING changes_stream ON delta_table.id = changes_stream.id
WHEN MATCHED THEN UPDATE SET value = changes_stream.value
WHEN NOT MATCHED THEN INSERT (id, value) VALUES (changes_stream.id, changes_stream.value)
```

**Example input / output:**

```text
-- Input stream data
INSERT INTO delta_table VALUES (1, 20)
INSERT INTO delta_table VALUES (2, 30)

-- Apply changes into existing Delta table
APPLY CHANGES INTO delta_table FROM changes_stream

-- Hand-written MERGE operation
MERGE INTO delta_table USING changes_stream ON delta_table.id = changes_stream.id
WHEN MATCHED THEN UPDATE SET value = changes_stream.value
WHEN NOT MATCHED THEN INSERT (id, value) VALUES (changes_stream.id, changes_stream.value)
```

**Data:**

```json
{
  "schema": {
    "fields": [
      {"name": "id", "type": "string"},
      {"name": "value", "type": "int"}
    ]
  }
}
```

**Flow:**

1. Input Data Stream
2. Process CDC Changes
3. DLT APPLY CHANGES INTO
4. Leverage Delta Lake'sTransactions
5. Consistent and AtomicOperations
6. Hand-written MERGEOperation
7. Manually Handle Transactions
8. Maintain Data IntegrityManually

