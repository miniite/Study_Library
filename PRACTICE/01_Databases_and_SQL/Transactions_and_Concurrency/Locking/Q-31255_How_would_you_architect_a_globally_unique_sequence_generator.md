---
type: "practice"
difficulty: "Deep"
---

# How would you architect a globally unique sequence generator that avoids the single-row lock bottleneck of a naive counter table?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Row Locks Two-phase Locking and Lock Footprint](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Row_Locks_Two-phase_Locking_and_Lock_Footprint.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a globally unique sequence generator that avoids the single-row lock bottleneck, we need to design a system that can efficiently generate monotonically increasing IDs across multiple nodes without resorting to locking mechanisms on a single row. The key is to leverage distributed techniques and ensure consistency through appropriate concurrency control strategies.

#### Triaging the Question

The question is essentially asking for a solution to generate unique sequence numbers in a distributed environment, ensuring no two nodes will generate the same ID simultaneously. This involves understanding how to handle write conflicts, ensuring data coherence across multiple instances, and optimizing for performance and scalability.

#### Assumptions
and First Principles
We are working with a distributed system where multiple nodes may need to request new sequence IDs.
The sequence generator must support high throughput while maintaining global uniqueness.
Each node can have partial visibility into the state of other nodes or rely on an external service for coordination.

#### Solution
Use a Distributed Counter System: One approach is to use a distributed counter mechanism, such as Apache ZooKeeper or Consul, where each node increments a shared counter atomically using a distributed consensus protocol like Raft.
Incremental Sequence Generation: Each time a node needs a new sequence ID, it fetches the current value from the distributed counter and increments it locally before returning the next available ID to the application layer.
Modular Design: Implement the sequence generator as a separate service or module that nodes can query for IDs. This ensures loose coupling between generating sequences and using them in applications.
Step-by-Step Solution
Initialize Distributed Counter Service:
Deploy a distributed counter service on multiple nodes, ensuring it can handle requests from any node.
Node Requests Sequence ID:
When a node needs a new sequence ID, it sends a request to the distributed counter service.
Counter Service Response:
The counter service receives the request and performs an atomic increment operation using a distributed consensus algorithm.
It then returns the incremented value (sequence ID) back to the requesting node.

#### Example
Implementation
Yes
No
Node Requests Sequence ID
Distributed Counter Service
Is Counter Ready?
Increment Counter and ReturnNew Value
Await Acknowledgement fromCounter Service
Code Example
PYTHON
class DistributedCounter:
    def __init__(self, counter_service_url):
        self.counter_service_url = counter_service_url

    def get_next_sequence_id(self):
        # Send a request to the distributed counter service
        response = requests.get(f"{self.counter_service_url}/next")
        if response.status_code == 200:
            return response.json()['sequence_id']
        else:
            raise Exception("Failed to retrieve sequence ID")

# Example Usage
counter_service_url = "http://counter-service.example.com"
distributor = DistributedCounter(counter_service_url)
next_sequence_id = distributor.get_next_sequence_id()
print(f"Next Sequence ID: {next_sequence_id}")

#### Closing Summary
and Common Mistakes/Follow-Ups

By using a distributed counter system, we can efficiently generate unique sequence IDs in a scalable manner without the need for single-row locks. However, it’s crucial to ensure that the distributed service is highly available and robust against failures.

#### Common Mistakes

Failing to handle network partitions correctly.
Not considering edge cases like retries or race conditions.
Ignoring the cost of maintaining a distributed system.

#### Follow-Ups

How would you handle situations where the counter service becomes unavailable?
What strategies can be used to ensure high availability and prevent data loss?

**Flow:**

1. Yes
2. No
3. Node Requests Sequence ID
4. Distributed Counter Service
5. Is Counter Ready?
6. Increment Counter and ReturnNew Value
7. Await Acknowledgement fromCounter Service

