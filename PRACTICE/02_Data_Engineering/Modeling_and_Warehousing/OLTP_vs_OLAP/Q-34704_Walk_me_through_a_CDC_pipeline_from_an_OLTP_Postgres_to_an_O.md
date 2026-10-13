---
type: "practice"
difficulty: "Deep"
---

# Walk me through a CDC pipeline from an OLTP Postgres to an OLAP warehouse — where do ordering, exactly-once and schema-drift problems bite you?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Feeding OLAP from OLTP](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/Feeding_OLAP_from_OLTP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a Change Data Capture (CDC) pipeline from an OLTP PostgreSQL database to an OLAP warehouse, there are several key challenges that can arise due to differences between OLTP and OLAP systems, particularly in terms of ordering, exactly-once processing, and schema drift. Understanding these issues is crucial for ensuring the integrity and reliability of data within your analytics system.

#### Triage
What the Question Is Asking

The question asks us to detail how common problems like ordering, exactly-once processing, and schema drift manifest when implementing a CDC pipeline from an OLTP database (PostgreSQL) to an OLAP warehouse. The interviewer is probing our understanding of the differences between these two types of databases and the challenges they present in a data integration scenario.

Problem Breakdown: Ordering

When integrating data from an OLTP system, order matters significantly because transactions are often time-sensitive and ordered based on when they were committed to the database. However, in an OLAP environment, there is less emphasis on transactional order, as it's more about aggregating large volumes of historical data.

Problem Breakdown: Exactly-Once Processing

Exactly-once processing ensures that each message is processed exactly once, which is critical for maintaining consistency and avoiding duplicate or lost updates. In a CDC pipeline, this can be challenging due to the asynchronous nature of data streaming and potential failures during transmission or processing.

Problem Breakdown: Schema Drift

Schema drift refers to changes in the structure of the database schema over time. While OLTP databases frequently undergo schema evolution to adapt to business needs, OLAP warehouses typically have more static schemas designed for long-term storage. Synchronizing these differing schemas can introduce significant complexity and potential data integrity issues.

#### First-Principles Solution
Ordering

To maintain order in a CDC pipeline from an OLTP database, we need to ensure that the changes are captured in the correct sequence as they occurred in the source system. This is particularly important for transactions that are time-dependent or have dependencies on other transactions. One approach is to use timestamps or sequence numbers embedded in the transaction data itself.

Exactly-Once Processing

Implementing exactly-once processing involves ensuring that each change event is processed only once, even if there are retries due to network failures or system downtime. This can be achieved using techniques such as:

Message Acknowledgment: Ensure messages are acknowledged upon successful processing and re-attempted if acknowledgment fails.
Idempotent Operations: Design your ETL processes to handle duplicate events gracefully without affecting the final state of the data.
Schema Drift

Handling schema drift requires mechanisms to map changes between the evolving OLTP schema and the static OLAP schema. This can be accomplished through:

Event Sourcing: Store all changes in a format that captures the "before" and "after" states, allowing for schema evolution without losing data integrity.
Delta Updates: Use incremental updates to apply only the necessary changes to the OLAP warehouse based on the current state of the source system.

#### Closing Summary

In summary, designing a CDC pipeline from an OLTP PostgreSQL database to an OLAP warehouse involves addressing several challenges, including maintaining transactional order, ensuring exactly-once processing, and managing schema drift. By understanding these issues at a fundamental level, we can design robust solutions that ensure data integrity and consistency.

#### Common Mistakes/Follow-Ups
Failing to Preserve Transaction Order: Not properly capturing the sequence of changes in the CDC pipeline.
Lack of Exactly-Once Guarantees: Inadequate error handling and recovery mechanisms leading to duplicate or missing updates.
Ignoring Schema Drift: Failing to implement mechanisms that adapt to evolving schemas, resulting in data integrity issues.

#### Follow-ups
might include specific technologies for implementing exactly-once processing (e.g., Kafka's Exactly-Once Semantics) or strategies for managing schema drift in a microservices environment.

**Comparison:**

| Order ID | Customer ID | Timestamp (Source)      | Sale Timestamp (Target) |
|----------|-------------|------------------------|------------------------|
| 9876     | 101         | 2023-10-01T09:00:00Z   | 2023-10-01T09:00:00Z   |

**Code:**

```python
# Example of using Python to acknowledge messages in an ETL pipeline
def process_message(message):
    try:
        # Process the message
        result = handle_data(message)

        # Acknowledge the message as processed
        ack(message.message_id)
    except Exception as e:
        # Handle exceptions and ensure idempotent processing
        log.error(f"Error processing message: {e}")
```

**Example input / output:**

```text
Input:
- Message ID: 12345
- Data: {"order_id": 9876, "customer_id": 101, "timestamp": "2023-10-01T09:00:00Z"}

Output:
- Processed data with timestamp adjustment for OLAP: {"order_id": 9876, "customer_id": 101, "sale_timestamp": "2023-10-01T09:00:00Z"}
```

**Data:**

```json
{
  "source_system": {
    "database_type": "PostgreSQL",
    "schema": [
      {"name": "orders", "columns": ["order_id", "customer_id", "timestamp"]}
    ]
  },
  "target_system": {
    "database_type": "OLAP Warehouse",
    "schema": [
      {"name": "sales_data", "columns": ["order_id", "customer_id", "sale_timestamp"]}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Transaction Occurs in OLTP
4. Is it a CDC Event?
5. Capture and Log Event
6. Handle Regular OLTPOperations
7. Acknowledge Event
8. Process Event in OLAPWarehouse
9. Update Schema if Necessary

