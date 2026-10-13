---
type: "practice"
difficulty: "Deep"
---

# You must guarantee a global unique invoice number with no gaps for compliance, in a high-concurrency system. Why do sequences/auto-increment leave gaps and how would you design a gapless counter safely?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Isolation Anomalies](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Isolation_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a globally unique and gapless invoice number in a high-concurrency system, we need to address why sequences or auto-increment alone are insufficient and then explore alternative strategies. Sequences/auto-increment can lead to gaps due to race conditions when multiple transactions try to increment the counter simultaneously. By leveraging distributed systems principles, we can design a robust solution that ensures no gaps in invoice numbers.

#### Triage
Understanding the Problem

The core of the question revolves around guaranteeing gapless invoice numbering while handling high concurrency. Sequences and auto-increment are prone to race conditions, which can result in gaps in the sequence. The interviewer is probing your understanding of concurrency issues, database transactions, and distributed systems concepts.

#### Assumptions
and Key Concepts
High Concurrency: Multiple requests for new invoices arriving simultaneously.
Gaps in Sequence Numbers: This occurs when multiple processes attempt to increment a shared counter at the same time.
Global Uniqueness: The invoice numbers must be unique across all systems and time.

#### First Principles Solution
Step 1: Recognize the Race Condition

In a traditional database with sequences or auto-increment, each transaction increments the counter. If multiple transactions attempt to increment the counter simultaneously, one of them might fail due to a concurrent update, leading to gaps in the sequence.

Step 2: Use Atomic Operations

To prevent race conditions, we need atomic operations that ensure only one process can increment the counter at any given time. This can be achieved through optimistic locking or pessimistic locking mechanisms like transactions with serializable isolation levels.

Step 3: Implement a Distributed Counter Strategy

A distributed system approach is needed to handle high concurrency and ensure global uniqueness. One common strategy is using a centralized service that acts as a coordinator for the invoice number generation.

Detailed Solution
Database Sequences/Auto-Increment: Traditional database sequences or auto-increment fields are not suitable due to their inherent race conditions.
Centralized Service Approach:
Create a separate, highly available service responsible for generating unique invoice numbers.
Use a distributed lock mechanism (e.g., Redis) to ensure atomicity when incrementing the counter.

#### Example
Using Redis for Distributed Counter
S3
S2
S1
Redis
DB
S3
S2
S1
Redis
DB
INCR invoice_number_key
+1, current_value=1000
INCR invoice_number_key
+1, current_value=1001
INCR invoice_number_key
+1, current_value=1002

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes
Failing to recognize the race condition in auto-increment/sequences or not understanding how distributed systems can mitigate these issues.

#### Follow-Ups
Discussing how to handle failures during counter updates, ensuring leader election if Redis fails, and considering performance implications of frequent locking.
Summary

To guarantee gapless invoice numbers in a high-concurrency system, we must avoid race conditions through atomic operations and utilize a distributed approach. A centralized service with distributed locks ensures that each increment operation is performed atomically and globally consistently.

**Comparison:**

```
Step	Action
1	Create a centralized service for invoice number generation.
2	Use Redis to store and increment the current invoice number atomically.
3	Implement distributed locks to ensure atomicity across multiple processes.
```

**Code:**

```python
from redis import StrictRedis

redis_client = StrictRedis(host='localhost', port=6379, db=0)

def generate_invoice_number():
    current_value = redis_client.incr('invoice_number_key')
    return current_value
```

**Example input / output:**

```text
Input:
- `generate_invoice_number()`

Output:
- Invoice Number (e.g., `1000`, `1001`, `1002`)
```

**Data:**

```json
{
  "invoice_number_key": {
    "type": "string",
    "description": "Key used for storing the current invoice number in Redis."
  }
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. Is Distributed Lock Available?
5. Generate Invoice Number
6. Return Invoice Number
7. Wait for Lock Release

