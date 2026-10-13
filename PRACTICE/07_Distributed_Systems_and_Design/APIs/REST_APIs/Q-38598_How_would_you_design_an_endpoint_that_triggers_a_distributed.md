---
type: "practice"
difficulty: "Expert"
---

# How would you design an endpoint that triggers a distributed operation across three services so that a failure halfway through doesn't leave the system inconsistent, given HTTP has no distributed transaction?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Distributed Operations Behind an Endpoint](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Distributed_Operations_Behind_an_Endpoint.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To ensure consistency and avoid leaving the system in an inconsistent state during a distributed operation across three services using REST APIs, we need to implement an idempotent and atomic approach. This involves coordinating the operations through a centralized service or by leveraging message brokers like Kafka, ensuring that partial failures are handled gracefully.

#### Triaging the Question

The question is asking how to design an endpoint that can coordinate and manage distributed operations across three services in such a way that if any step fails halfway through, the system remains consistent. The key challenge here lies in the fact that HTTP does not support distributed transactions natively, making it difficult to ensure atomicity.

#### Assumptions
Each of the three services has a well-defined API endpoint.
We need to trigger an operation across these three services sequentially or in parallel.
Partial failures are possible and must be handled gracefully.
The system should remain consistent even if some steps fail.

#### First Principles Solution
Step 1: Coordination Service

We can introduce a centralized coordination service (CDS) that acts as the single point of truth for orchestrating these operations. This CDS will handle the following responsibilities:

Track the state of each operation.
Ensure idempotency: If an operation is called multiple times, it should not affect the system more than once.
Handle retries and backoff strategies to manage partial failures.
Step 2: Message Broker

Using a message broker like Apache Kafka or Amazon SQS can help in managing asynchronous communication between services. The CDS will publish messages for each service operation, allowing for retry mechanisms if necessary without causing infinite loops.

Step 3: Service A, B, and C Endpoints

Each of the three services (A, B, and C) should have endpoints that:

Accept a unique identifier for the operation.
Validate and process the request.
Return an acknowledgment or error response.
Step 4: Orchestrating Operations

The coordination service will orchestrate the operations as follows:

Request Reception: The CDS receives a request to trigger the distributed operation.
Message Publishing: The CDS publishes messages to Service A, B, and C with unique identifiers for each step.
Acknowledgment Handling:
If all services acknowledge successful processing, the CDS marks the operation as completed.
If any service fails, the CDS handles the failure by logging it, rolling back partially completed steps if necessary, or retrying with exponential backoff.

#### Cons
istency Check: The CDS ensures that after all messages are processed, a final consistency check is performed to ensure no state inconsistencies.
Step 5: Example Workflow
PLAINTEXT
1. User sends request to CDS
2. CDS publishes message to Service A (id: 1)
3. Service A processes -> ACK to CDS
4. CDS publishes message to Service B (id: 1)
5. Service B processes -> ACK to CDS
6. CDS publishes message to Service C (id: 1)
7. Service C processes -> ACK to CDS
8. CDS marks operation as completed and performs final consistency check.
Step 6: Handling Failures

If any service fails, the CDS:

Logs the failure and re-publishes the failed message with exponential backoff.
Rolls back partially completed steps if necessary.

#### Closing Summary
+ Common Mistakes/Follow-ups:

By leveraging a centralized coordination service and a message broker, we can achieve consistency across distributed services. Key points to remember include ensuring idempotency, handling partial failures gracefully, and performing final consistency checks.

#### Common mistakes
often involve:

Ignoring the need for a coordinated approach.
Not implementing proper error handling and retries.
Failing to perform thorough testing of edge cases and failure scenarios.

#### Follow-ups
could include discussing specific implementation details like choosing between synchronous or asynchronous processing, integrating with existing infrastructure, and optimizing performance under high loads.

Triaging:

This solution balances complexity with robustness while ensuring that the system remains consistent in the face of partial failures. It leverages modern patterns like service orchestration and message brokers to achieve distributed consistency without relying on HTTP transactions.

**Comparison:**

```
Operation ID	Status	Services
1	Completed	A: Success, B: Failure, C: Success
```

**Code:**

```python
# Example CDS code for publishing messages
import kafka

kafka_client = kafka.KafkaProducer(bootstrap_servers=['localhost:9092'])
message_id = '1'

# Publish to Service A
kafka_client.send('service_a_topic', key=message_id.encode(), value=b'operation_start')

# Receive ACK from Service A
# Handle ACK and send to next service
```

**Example input / output:**

```text
Input:
- User sends a request to CDS to trigger distributed operation.

Output:
- CDS publishes messages to Service A, B, and C.
- Each service processes the message and responds with ACK or NACK.
- CDS tracks status and handles failures by retrying with backoff.
- Final consistency check is performed after all services are processed.
```

**Data:**

```json
{
  "operations": {
    "id": 1,
    "status": "completed",
    "services": [
      {"service_name": "ServiceA", "status": "success"},
      {"service_name": "ServiceB", "status": "failure"},
      {"service_name": "ServiceC", "status": "success"}
    ]
  }
}
```

**Flow:**

1. User Sends Request
2. CDS Receives Request
3. CDS Publishes Message toService A
4. Service A Processes & ACKs
5. CDS Acknowledges Service A
6. CDS Publishes Message toService B
7. Service B Processes & NACKsdue to Error
8. CDS Handles Failure, Retrieswith Backoff
9. CDS Publishes Message toService C
10. Service C Processes & ACKs
11. CDS Acknowledges Service C
12. CDS Marks Operation asCompleted
13. Final Consistency CheckPerformed

