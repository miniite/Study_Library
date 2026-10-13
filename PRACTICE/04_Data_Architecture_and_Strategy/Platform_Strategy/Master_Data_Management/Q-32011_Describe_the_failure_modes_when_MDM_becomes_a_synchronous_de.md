---
type: "practice"
difficulty: "Expert"
---

# Describe the failure modes when MDM becomes a synchronous dependency in the write path of an order/payment flow — how do you keep it out of the critical path while still enforcing uniqueness?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ MDM for Batch and Streaming](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/MDM_for_Batch_and_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're exploring how Master Data Management (MDM) can become a bottleneck in real-time transaction processing, particularly when orders or payments are being created. The core issue is to ensure that MDM operations do not block the write path of an order/payment flow while still maintaining uniqueness and data integrity.

#### Triaging the Question

The question probes understanding of how to balance the need for accurate, consistent master data with the requirement for low-latency transaction processing. Specifically, it challenges us to consider:

Synchronization Issues: How MDM operations might delay or block the order/payment flow.
Uniqueness Enforcement: Ensuring that records are unique without introducing latency.
Key Assumptions
The system processes orders/payments in real-time and requires immediate completion.
MDM is critical for maintaining data integrity, such as ensuring customer and product information is accurate.
There's a need to enforce uniqueness constraints on certain fields (e.g., unique customer IDs).

#### First-Principles Solution

To address these challenges, we can implement an asynchronous approach to MDM updates while still enforcing uniqueness through carefully crafted validation mechanisms.

Asynchronous Data Processing:

Introduce a background worker that handles MDM operations independently of the order/payment write path.
The background worker should process incoming data in batches or asynchronously based on a queue.

Validation and Deduplication:

Use a cache or database with strong consistency guarantees to validate uniqueness before writing an order/payment.
This ensures that even if MDM operations take time, the immediate write path remains smooth.

Event-Driven Architecture:

Leverage event-driven architecture where orders/payments trigger events for background MDM updates.
Events can include pre-validation checks to ensure data is clean before sending it through to MDM workers.

#### Implementation
Steps

Design the Background Worker:

Yes
No
Order/Payment Received
Is Unique?
Write Order/Payment
Process through MDM Queue
Update MDMCache/Databases
Revalidate in Background ifNecessary
Finalize Order/Payment inDatabase

Use Caching for Uniqueness Checks:

Implement a caching layer (e.g., Redis) to store and quickly check uniqueness.
If the cache reports a conflict, reject the order/payment immediately.

Event-Driven Triggers:

Yes
Order/Payment Received
Trigger MDM Event?
Send to MDM Worker
MDM Worker Processes Data
Cache Updated with New Data
Revalidate in Cache ifNecessary

#### Closing Summary
and Common Mistakes

To summarize, the key is to decouple MDM operations from the critical write path through asynchronous processing. By using caches for immediate validation and event-driven architecture, we can maintain both data integrity and low-latency transactions.

#### Common mistakes
include:

Failing to implement proper deduplication checks, leading to duplicate records.
Not using background workers, causing real-time performance bottlenecks.
Overlooking the need for strong consistency in cache mechanisms during high-concurrency scenarios.

**Comparison:**

```
Input	Cache Check	Action
Order	Pass	Write
Dup. Order	Fail	Reject
```

**Code:**

```python
# Example of a simple validation function using Redis as a cache
import redis

r = redis.Redis(host='localhost', port=6379, db=0)

def validate_uniqueness(data):
    key = f'uniqueness:{data["key_field"]}'
    if not r.exists(key):
        r.setex(key, 10, 'true')  # Set expiration time to prevent false positives
        return True
    else:
        return False

# Example of sending data to a background worker
def send_to_mdm_worker(data):
    # Code to queue the data for processing in MDM
    pass
```

**Example input / output:**

```text
Input: Order with customer ID '098765'
Output:
- Validation check in cache passes (order written to database)
- Background worker processes MDM updates

Input: Duplicate order with same customer ID
Output:
- Uniqueness check fails, write to DB rejected immediately
```

**Data:**

```json
{
  "order_id": "123456",
  "customer_id": "098765",
  "product_id": "987654"
}
```

**Flow:**

1. Yes
2. No
3. Order/Payment Received
4. Is Unique?
5. Write Order/Payment
6. Process through MDM Queue
7. Update MDMCache/Databases
8. Revalidate in Background ifNecessary
9. Finalize Order/Payment inDatabase

