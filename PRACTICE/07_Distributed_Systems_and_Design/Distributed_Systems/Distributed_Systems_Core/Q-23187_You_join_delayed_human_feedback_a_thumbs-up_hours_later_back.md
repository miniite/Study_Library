---
type: "practice"
difficulty: "Intermediate"
---

# You join delayed human feedback (a thumbs-up hours later) back to the original LLM generation across a high-throughput fleet. Explain the join-key, storage, and windowing design and how you handle feedback that arrives after your retention window.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Load Balancing Stateful Inference](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/Load_Balancing_Stateful_Inference.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of incorporating delayed human feedback into an LLM generation system across a high-throughput fleet, we need to focus on three key aspects: the join-key design, storage strategy, and handling feedback outside the retention window. The goal is to ensure that our system can efficiently process and apply this feedback without overwhelming resources or losing valuable data.

Firstly, let’s define what we are trying to achieve:

Join-Key Design: This involves creating a unique identifier (or key) that links the original LLM generation request with its corresponding delayed response.
Storage Strategy: We need an efficient way to store and retrieve these join keys while handling high throughput.
Handling Feedback Outside Retention Window: We must decide how to deal with feedback that arrives after our system has already processed the request.

#### Triaging the Question

The core of this question lies in designing a robust, scalable system for delayed human feedback. The interviewer is likely probing your understanding of distributed systems, data integrity, and handling edge cases such as late-arriving data.

#### Assumptions
High Throughput: The LLM generation system processes many requests per second.
Delayed Feedback: Human reviewers provide feedback hours after the initial request, which needs to be incorporated into future generations.
Retention Window: There is a finite period during which delayed feedback can still be useful.
Step-by-Step Solution

Join-Key Design

The join key should uniquely identify each LLM generation request and its corresponding response. A common approach is to use the request ID or a combination of timestamp and user ID.
For example, request_id could be a UUID or a timestamp-based unique identifier.

Storage Strategy

Use a distributed database like Cassandra or DynamoDB that can handle high writes and reads.
Store both the original LLM request data and the delayed feedback using this join key.
Implement partitioning to distribute the load evenly across multiple nodes, ensuring low latency for read and write operations.

Handling Feedback Outside Retention Window

If the retention window is set to T hours and a piece of feedback arrives after T+1 hours, we need to decide if it’s still valuable.
One approach is to allow late feedback but give it a lower weight or consider it as an outlier. Alternatively, you could implement a system that re-runs the generation process with the latest available data up to the point where the feedback arrives.

#### Closing Summary
and Common Mistakes

#### Common Mistakes
Overlooking the need for robust error handling and ensuring data consistency across distributed systems.

#### Follow-Ups
Discussing how to optimize storage costs, dealing with network partitions, and ensuring that the system remains performant under high load.

**Comparison:**

```
Table Name	Primary Key	Columns
requests	request_id	data (map)
feedback	request_id	feedback_data (map)
```

**Code:**

```python
# Example code snippet for storing join keys in a distributed database

import uuid
from cassandra.cluster import Cluster

cluster = Cluster(['127.0.0.1'])
session = cluster.connect()

def store_request(request_data):
    request_id = str(uuid.uuid4())
    session.execute(
        """
        INSERT INTO requests (request_id, data) VALUES (%s, %s)
        """,
        (request_id, request_data)
    )
    return request_id

# Example of storing delayed feedback
def store_feedback(feedback_data, request_id):
    session.execute(
        """
        INSERT INTO feedback (request_id, feedback_data) VALUES (%s, %s)
        """,
        (request_id, feedback_data)
    )
```

**Example input / output:**

```text
Input:
- Request data for a LLM generation request
- Delayed feedback data

Output:
- Stored request and feedback in the database with corresponding join keys
- Example of handling late feedback by re-running the process up to the point where the feedback arrives.
```

**Data:**

```json
{
  "requests_table": {
    "primary_key": ["request_id"],
    "columns": [
      {"name": "request_id", "type": "text"},
      {"name": "data", "type": "map"}
    ]
  },

  "feedback_table": {
    "primary_key": ["request_id"],
    "columns": [
      {"name": "request_id", "type": "text"},
      {"name": "feedback_data", "type": "map"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Generate Request
4. Is Feedback Late?
5. Re-run Generation with LatestData
6. Store Feedback

