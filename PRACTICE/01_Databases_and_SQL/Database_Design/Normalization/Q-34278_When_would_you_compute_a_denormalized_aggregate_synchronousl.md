---
type: "practice"
difficulty: "Deep"
---

# When would you compute a denormalized aggregate synchronously in the write path versus asynchronously in a downstream streaming job?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Denormalization and Locking](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/Denormalization_and_Locking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address when to compute a denormalized aggregate synchronously or asynchronously, let's first understand what we're dealing with. Synchronous computation happens immediately upon receiving a write request, while asynchronous computation happens in a downstream streaming job, which processes data independently of the original writes. The choice between these approaches depends on several factors such as system architecture, performance requirements, and data integrity needs.

#### Triage

The question is probing your understanding of when to use synchronous vs. asynchronous processing for denormalized aggregates. This involves considering trade-offs like immediate availability versus resource utilization and latency. Key assumptions include:

We are working within a database or ETL pipeline context.
Denormalization is required for performance reasons, such as improving read speeds.

#### First Principles Solution
Synchronous Computation
Immediate Availability: Synchronous computation ensures that the aggregate data is available immediately upon write. This is critical when real-time insights are needed.
Simplicity in Logic: The logic for updating denormalized aggregates can be simpler and easier to reason about, as it happens at the point of origin.
Resource Utilization: Synchronous processing can lead to higher resource utilization during writes, potentially impacting performance.
Asynchronous Computation
Reduced Write Latency: Asynchronous computation offloads the denormalization task, reducing write latency for the original data sources.
Scalability: By processing in a separate job, you can scale resources independently of read-heavy operations, which is beneficial in large-scale systems.
Resource Management: You can optimize resource allocation by focusing on reads during high-write periods and vice versa.

#### Closing Summary
+ Common Mistakes

Synchronous denormalization should be used when real-time access to aggregated data is crucial and the system has sufficient resources to handle the additional load. Asynchronous processing is more suitable for scenarios where write-heavy operations need to remain responsive, and the benefits of reduced latency outweigh the delays in updating aggregates.

#### Common mistakes
include:

Overlooking the impact on resource utilization during peak write times when choosing synchronous.
Failing to consider the added complexity and potential data consistency issues that asynchronous approaches might introduce.

**Comparison:**

```
Synchronous	Asynchronous
Immediate availability of aggregated data	Reduced write latency
Simpler logic and easier to reason about	Independent scalability and resource management
```

**Code:**

```python
# Example function for synchronous denormalization
def sync_denormalize(data):
    # Immediate update of denormalized data
    aggregate = calculate_aggregate(data)
    update_database(aggregate)

# Example setup for asynchronous denormalization using a streaming job
import time

def async_denormalize(stream_data, output_queue):
    while True:
        data = stream_data.get()
        if data is not None:
            aggregate = calculate_aggregate(data)
            output_queue.put(aggregate)
            time.sleep(0.1)  # Simulate processing delay
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "write_request": {
    "id": 123,
    "data": {
      "name": "John Doe",
      "orders": [
        { "product_id": 1, "quantity": 5 },
        { "product_id": 2, "quantity": 3 }
      ]
    }
  }
}

Output (Synchronous):

JSON
Copy
{
  "aggregate_data": {
    "name": "John Doe",
    "total_orders": 8,
    "average_quantity_per_order": 4.0
  }
}

Output (Asynchronous):

JSON
Copy
{
  "async_output_queue": [
    {
      "id": 123,
      "aggregate_data": {
        "name": "John Doe",
        "total_orders": 8,
        "average_quantity_per_order": 4.0
      }
    },
    ...
  ]
}
```

**Data:**

```json
{
  "write_request": {
    "id": 123,
    "data": {
      "name": "John Doe",
      "orders": [
        { "product_id": 1, "quantity": 5 },
        { "product_id": 2, "quantity": 3 }
      ]
    }
  }
}
```

**Flow:**

1. Asynchronous
2. Synchronous
3. Write Request
4. Synchronous?
5. Compute Aggregate, UpdateDB
6. Asynchronous?
7. Process in Streaming Job
8. Wait for Processing, thenCompute & Store Aggregates
9. Write Latency Increased
10. Write Latency Reduced
11. Independent Scalability

