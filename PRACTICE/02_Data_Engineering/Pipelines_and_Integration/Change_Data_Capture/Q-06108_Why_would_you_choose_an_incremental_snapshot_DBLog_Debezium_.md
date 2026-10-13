---
type: "practice"
difficulty: "Expert"
---

# Why would you choose an incremental snapshot (DBLog/Debezium) over a blocking snapshot for a 2 TB table and what watermarking makes it correct?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Debezium Transforms and Snapshots](<../../../../02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Debezium_Transforms_and_Snapshots.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To summarize, we're evaluating between an incremental snapshot approach using tools like DBLog or Debezium and a blocking snapshot (full export) for capturing changes in a 2 TB table. The key decision factor is whether to use watermarking with the incremental capture method. Let's dive into why you might choose incremental snapshots over full exports, and what exactly watermarking does.

Incremental snapshots are favored when real-time data freshness is critical but complete historical data isn't necessary. Tools like DBLog or Debezium can track changes in a database, making them highly efficient for large datasets without the need to export all historical records at once. This approach minimizes resource usage and reduces latency compared to blocking snapshots.

#### Triage
The question primarily seeks an understanding of why incremental CDC is preferred over full table exports, especially in terms of performance and operational constraints. Key assumptions include the size of the dataset (2 TB), the need for near-real-time updates, and the environment's ability to handle continuous change tracking. We'll break down how watermarking ensures the correctness of the incremental approach.

Step-by-step reasoning from first principles:

Understanding Incremental CDC: Incremental Change Data Capture (CDC) captures only the changes made to a dataset over time. This is often achieved using tools like DBLog or Debezium, which can hook into database transactions and log changes as they occur.
Incremental CDC vs Blocking Snapshots:
Performance: Full table exports are resource-intensive and take significant time, making them unsuitable for real-time applications. Incremental CDC is more efficient because it only processes new or updated records.
Latency: With full exports, latency is high due to the need to pause operations while data is exported. Incremental CDC can maintain low-latency updates by processing changes as they happen.
Watermarking for Correctness:
Definition: Watermarking is a mechanism that records the timestamp of the latest processed event. It helps ensure that all changes up to a certain point in time have been captured and processed correctly.

#### Implementation
Tools like Debezium automatically manage watermarks by tracking the last transaction ID or sequence number, ensuring only changes after this point are processed.

#### Example
Suppose you start with a table at 10:00 AM. Watermarking ensures that if you set the watermark to 10:30 AM, all updates made between 10:00 AM and 10:30 AM will be captured, but no older records are processed.

#### Closing Summary
+ Common Mistakes/Follow-ups: Incremental CDC with watermarking is ideal for large datasets where real-time data freshness matters more than historical completeness. It minimizes resource usage and reduces latency, making it a better choice in most production environments compared to full table exports. Key challenges include ensuring the accuracy of the timestamp tracking mechanism and handling edge cases like transaction rollbacks or failures.

#### Common mistakes
include not properly setting up and maintaining watermarks, leading to data inconsistencies. Follow-up questions might involve discussing how to handle specific failure modes or optimizing watermarking strategies for different use cases.

**Comparison:**

```
Timestamp	Action	Data
2023-10-01T10:05	Insert	Order ID 1
2023-10-01T10:15	Update	Order ID 1 (updated)
2023-10-01T10:45	Delete	Order ID 1
```

**Code:**

```python
# Example of a simple Python function to simulate watermark handling
def process_changes(changes, watermark):
    processed_records = []
    for change in changes:
        if change.timestamp >= watermark:
            processed_records.append(change)
    return processed_records

changes = [
    {"timestamp": 10:30, "data": "record_1"},
    {"timestamp": 10:45, "data": "record_2"},
    {"timestamp": 10:15, "data": "record_3"}
]

watermark = 10:30
processed_records = process_changes(changes, watermark)
print(processed_records) # Output: [{'timestamp': 10:30, 'data': 'record_1'}, {'timestamp': 10:45, 'data': 'record_2'}]
```

**Example input / output:**

```

Input: A 2 TB table with ongoing changes.

Output:

Initial snapshot at 10:00 AM.
Watermark set to 10:30 AM.
Changes captured from 10:00 AM to 10:30 AM, ignoring older data.

Dry-run:

PYTHON
Copy
changes = [
    {"timestamp": "2023-10-01T10:05", "data": "insert_order_1"},
    {"timestamp": "2023-10-01T10:15", "data": "update_order_1"},
    {"timestamp": "2023-10-01T10:45", "data": "delete_order_1"}
]
watermark = "2023-10-01T10:30"
processed_changes = process_changes(changes, watermark)
print(processed_changes) # Output: [{'timestamp': '2023-10-01T10:15', 'data': 'update_order_1'}, {'timestamp': '2023-10-01T10:45', 'data': 'delete_order_1'}]
```

**Data:**

```json
{
    "table_name": "customer_orders",
    "size": "2TB",
    "incremental_changes": [
        {"timestamp": "2023-10-01T10:00", "data": "insert_order_1"},
        {"timestamp": "2023-10-01T10:15", "data": "update_order_1"},
        {"timestamp": "2023-10-01T10:45", "data": "delete_order_1"}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is watermark set?
5. Process changes from start towatermark
6. Set watermark and continueprocessing
7. Output processed records
8. Continue processing with newwatermark

