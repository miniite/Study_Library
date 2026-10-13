---
type: "practice"
difficulty: "Deep"
---

# Explain how change data capture (CDC) works and why it's often better than polling for building derived data stores. What ordering and at-least-once guarantees does log-based CDC give you?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Keeping Indexes in Sync via CDC](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Keeping_Indexes_in_Sync_via_CDC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Change Data Capture (CDC) is a critical technique in distributed systems that allows us to efficiently capture changes made to database records without needing to poll them repeatedly. It's often preferred over polling because it offers lower latency, more reliable data, and reduces the load on databases. This answer will explain how CDC works, why it’s better than polling, and discuss the ordering and at-least-once guarantees provided by log-based CDC.

Firstly, let's break down what we're dealing with:

Polling: It involves periodically querying a database or service to check for updates.
CDC: It captures changes directly from the database as they happen. This is typically done using the database’s transaction logs or through built-in CDC features.

Now, let's dive into how it works:

How CDC Works
Database Transaction Logs:
Most databases maintain a log of every operation (INSERT, UPDATE, DELETE) performed on them.
CDC Process:
A CDC service reads these logs in real-time or near-real-time.
It processes the records to extract meaningful data changes.
These processed records are then written into an event stream or directly into a derived data store.
Why CDC is Better than Polling
Latency:
Polling introduces significant latency as it requires waiting for the next poll interval, even if no updates have been made.
CDC captures changes immediately after they occur, reducing this delay to near-zero.
Load on Databases:
Frequent polling can place a heavy load on databases by constantly querying them.
CDC reduces this load because it only processes necessary data changes.
Data Consistency:
Polling might miss updates that occur between polls, leading to stale or inconsistent data.
CDC ensures that all changes are captured and processed without missing any.
Ordering and At-Least-Once Guarantees

When using log-based CDC, you can rely on specific ordering and at-least-once guarantees:

Ordering:
Log entries in a database typically have timestamps or sequence numbers.
These ensure that changes are captured and processed in the order they occurred.
At-Least-Once Processing:
CDC services often implement mechanisms to ensure each change is processed at least once, even if there are failures during processing.

#### Example Scenario

Suppose we have a database table orders with a primary key order_id. Here’s how CDC might work:

A new order is inserted into the orders table.
The database writes an entry to its transaction log indicating this insert operation.
A CDC service reads from the log and extracts the necessary information (e.g., order_id, customer_name, etc.).
These details are then stored in a derived data store like Apache Kafka or Amazon Kinesis.

#### Closing Summary

In summary, CDC provides significant advantages over polling by reducing latency, minimizing database load, and ensuring consistent data processing. By leveraging the built-in transaction logs of databases, log-based CDC can guarantee ordered and at-least-once processing, making it a robust solution for building derived data stores.

#### Common Mistakes
and Follow-Ups
Failing to Handle Duplicates: Ensure that your system is designed to handle duplicate records in case of reprocessing.
Latency Sensitivity: Be aware of the potential latency introduced by CDC services, especially if processing times are high.
Scalability Considerations: Understand how CDC will scale with increasing data volumes and complexity.

**Comparison:**

```
Log Entry	Type	Table	Order ID	Customer Name
1	INSERT	orders	123456	John Doe
```

**Code:**

```python
# Example Python pseudo-code for capturing changes from a log
def capture_changes_from_log(log):
    for entry in log:
        # Process the entry to extract meaningful data
        process_entry(entry)

def process_entry(entry):
    # Extract relevant fields
    order_id = entry['order_id']
    customer_name = entry['customer_name']

    # Store processed data into a derived store (e.g., Kafka topic)
    store_in_kafka(order_id, customer_name)
```

**Example input / output:**

```text
Input:
[{
  "timestamp": 1628407588,
  "type": "INSERT",
  "table": "orders",
  "order_id": 123456,
  "customer_name": "John Doe"
}]

Output (to Kafka):
Order: {"order_id": 123456, "customer_name": "John Doe"}
```

**Data:**

```json
{
  "log_entry": {
    "timestamp": 1628407588,
    "type": "INSERT",
    "table": "orders",
    "order_id": 123456,
    "customer_name": "John Doe"
  }
}
```

**Flow:**

1. Yes
2. No
3. Database Transaction
4. Log Entry Exists?
5. Read Log Entry
6. Extract Data
7. Store in Derived Store (Kafka)
8. No New Changes

