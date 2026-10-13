---
type: "practice"
difficulty: "Deep"
---

# How do you measure and reduce write amplification when one logical update touches five denormalized copies across your platform?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Denormalization and Locking](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/Denormalization_and_Locking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Write amplification is a critical performance metric in distributed systems, especially those employing denormalization to improve read performance. To measure and reduce write amplification when one logical update touches five denormalized copies across your platform, we need to understand the underlying data structures, the write patterns, and implement strategies like optimistic locking or incremental updates.

#### Triaging the Question

The question revolves around optimizing the write operations in a distributed system where multiple denormalized copies of data are involved. The core issue is that each update operation incurs overhead as it needs to propagate changes across several copies, leading to high write amplification. We need to measure this effect and then devise strategies to mitigate it.

#### First-Principles Solution

Understand the Data Model:

Identify how data is denormalized.
Determine which updates will affect multiple copies.

Measure Current Write Amplification:

Use metrics such as the ratio of write operations to logical updates.
Collect and analyze logs from your system to quantify this metric.

Optimizing Writes:

Implement optimistic locking to minimize redundant writes.
Use incremental or differential updates where feasible.

#### Cons
ider using event sourcing or a message queue to decouple write propagation.

Monitoring and Feedback Loops:

Continuously monitor write performance and adjust strategies as needed.
Introduce automated tests that simulate high update rates.
Detailed Solution
Step 1: Understand the Data Model

Assume we have an application where user profiles are denormalized to multiple tables for faster read operations. Each profile might exist in five different forms:

Basic Info Table
Preferences Table
Activity Stream
Notification Queue
User Feedback

When a user updates their email address, this change should be reflected across all these tables.

Step 2: Measure Current Write Amplification

Write amplification can be quantified by the ratio of write operations to logical updates. For instance:

CODE
Write Amplification = Total Write Operations / Logical Updates

To gather this data, we need:

Logs from database writes.
Metrics on update frequency for each denormalized copy.

#### Example
Metrics:

PLAINTEXT
Logical Update Rate: 100 updates per second
Total Write Operations: 500 write operations per second (across all copies)
Write Amplification = 500 / 100 = 5.0
Step 3: Optimizing Writes

Optimistic Locking:

Use versioning or timestamps to ensure data consistency.
If a write operation fails due to concurrent updates, retry the transaction.
PYTHON
def update_user_email(email):
    user_version = get_user_version(user_id)

    # Perform read and update in one transaction
    with database.transaction():
        new_version = increment_version(user_version)  # Assume version increments by 1 on success
        if not set_user_email(user_id, email, new_version):
            return False

    return True

Incremental Updates:

Instead of updating all copies in one go, update only the modified fields.

#### Example
Schema:

SQL
CREATE TABLE Profile (
    id INT PRIMARY KEY,
    basic_info JSONB,
    preferences JSONB,
    activity_stream JSONB,
    notification_queue JSONB,
    user_feedback JSONB
);

When an email is updated, only basic_info needs to be changed:

SQL
UPDATE Profile SET basic_info = jsonb_set(basic_info, '{email}', 'new_email') WHERE id = 123;

Message Queues:

Use a message broker like Kafka or RabbitMQ to handle update propagation asynchronously.

#### Example
Workflow:

PLAINTEXT
User Update -> Database Write -> Message Queue -> Propagation Service -> Update Copies
Step 4: Monitoring and Feedback Loops

Implement continuous monitoring of write operations and logical updates. Use tools like Prometheus, Grafana for visualizing metrics.

Automated Tests:

Simulate high update rates to detect bottlenecks.
Introduce stress tests to identify performance limits.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By measuring write amplification and optimizing writes through strategies like optimistic locking, incremental updates, and message queues, we can significantly reduce the overhead of write operations in a denormalized system. However, it's crucial to monitor these changes continuously to ensure they are effective.

#### Common Mistakes

Overlooking the need for versioning or optimistic locking.
Failing to measure and optimize based on real-world metrics.
Not considering asynchronous updates which can introduce complexity but improve performance.

#### Follow-Ups

Discuss trade-offs between consistency and availability in distributed systems.
Explore advanced techniques like multi-version concurrency control (MVCC) if you're targeting high transaction volumes.

**Comparison:**

```
Metric	Value
Logical Update Rate	100 updates/sec
Total Write Operations	500 write ops/sec
Write Amplification	5.0
```

**Code:**

```python
def update_user_email(email):
    user_version = get_user_version(user_id)

    # Perform read and update in one transaction
    with database.transaction():
        new_version = increment_version(user_version)  # Assume version increments by 1 on success
        if not set_user_email(user_id, email, new_version):
            return False

    return True

# Example SQL for incremental updates
UPDATE Profile SET basic_info = jsonb_set(basic_info, '{email}', 'new_email') WHERE id = 123;
```

**Example input / output:**

```

Example of a high update rate test:

Input: 100 updates per second to basic_info, preferences.
Output: Measure the total number of write operations.
PLAINTEXT
Copy
Logical Update Rate: 100
Total Write Operations: 500 (across all copies)
Write Amplification = 5.0
```

**Data:**

```sql
CREATE TABLE Profile (
    id INT PRIMARY KEY,
    basic_info JSONB,
    preferences JSONB,
    activity_stream JSONB,
    notification_queue JSONB,
    user_feedback JSONB
);
```

**Flow:**

1. Yes
2. No
3. User Update
4. Database Write
5. Is Version Conflicted?
6. Retry Transaction
7. Message Queue
8. Propagation Service
9. Update Copies

