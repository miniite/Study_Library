---
type: "practice"
difficulty: "Expert"
---

# A skeptical staff engineer says CDC-based outbox is enough and you don't need application idempotency. Construct the scenario where CDC still delivers duplicates downstream and idempotency remains mandatory.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Idempotency and Schema Evolution](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Idempotency_and_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the skepticism about using a Change Data Capture (CDC)-based outbox as sufficient for ensuring exactly-once message delivery semantics, we need to consider real-world complexities in distributed systems and data processing pipelines. The core issue is that while CDC can capture changes efficiently, it does not inherently guarantee idempotency at the application level. Let's construct a scenario where even with an outbox mechanism, duplicates could still be delivered downstream, making application-level idempotency crucial.

Firstly, let’s define our terms:

CDC-based Outbox: A pattern where changes to the database are captured and stored in a message queue or a dedicated table before being committed.
Idempotency: Ensuring that a particular operation can be called multiple times without changing the outcome beyond the first call.

Now, let's walk through the scenario step-by-step:

Scenario Setup:

We have an application that performs transactions on a database using CDC-based outbox to capture and queue these changes.
The application uses asynchronous messaging for some operations but not all.
There is a downstream service that processes messages from the outbox.

Problem Scenario:

Imagine a scenario where there’s a network partition or a system crash during message processing.
A transaction might be partially processed and then the system recovers, leading to duplicate messages being sent.

#### Cons
ider an operation like updating a user's address; if this operation is retried due to a transient error, it could result in multiple updates.

Key Issues:

Network Partition: During a network partition, some messages might be re-sent after the system recovers.
System Crash: If a downstream service processes a message and then crashes before fully committing the transaction, when the service restarts, it might process the same message again.

CDC Limitations:

CDC alone does not provide guarantees about delivery or processing order.
It captures changes but doesn’t ensure that these changes are processed exactly once by downstream systems.

Idempotency Requirement:

To handle such scenarios, the application must implement idempotent operations. This means that an operation can be safely retried without changing the outcome beyond the first call.
For example, if updating a user's address is idempotent, subsequent updates will not alter the state more than the initial update.

#### Implementation
Example:

Use message deduplication techniques in the outbox to prevent re-processing of already handled messages.
Implement unique identifiers or sequence numbers for each operation and check them before performing the operation again.
In our example, ensure that updating a user's address is idempotent by checking if the address has already been updated.

**Comparison:**

| User ID | Current Sequence Number |
|---------|------------------------|
| 123     | 456                    |

**Code:**

```python
# Example Python code to implement idempotency in an update operation

def update_address(user_id, new_address):
    # Check if the address is already up-to-date using a unique sequence number or hash
    current_sequence = get_current_sequence_number(user_id)
    if current_sequence != get_hash(new_address):
        set_sequence_number(user_id, get_hash(new_address))
        save_new_address(user_id, new_address)

# Example database operations
def get_current_sequence_number(user_id):
    return db.execute("SELECT sequence FROM addresses WHERE user_id = %s", (user_id,))

def set_sequence_number(user_id, new_sequence):
    db.execute("UPDATE addresses SET sequence = %s WHERE user_id = %s", (new_sequence, user_id))

def save_new_address(user_id, new_address):
    db.execute("UPDATE addresses SET address = %s WHERE user_id = %s", (new_address, user_id))
```

**Example input / output:**

```text
Input:
- User with ID 123 has their address updated to 'New York' in the database.

Output:
- Sequence number for user 123 is checked and set if necessary.
- Address update operation performed only once, ensuring idempotency.
```

**Data:**

```json
{
  "user_id": "123",
  "current_sequence_number": 456,
  "address_to_update": "New York"
}
```

**Flow:**

1. Yes
2. Satisfied
3. Not satisfied
4. No
5. Transaction
6. Database Change
7. Is the change in outbox?
8. Check idempotency condition
9. Commit to database, updatesequence number
10. Retry or log failure
11. Capture and queue in outbox

