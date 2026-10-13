---
type: "practice"
difficulty: "Expert"
---

# What's your strategy for architecting a multi-region write path when your business can't tolerate the latency of synchronous cross-region consistency?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Compute Placement Strategy](<../../../../06_Cloud_Computing/Cloud_Architecture/Architecture_Strategy/Compute_Placement_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of architecting a multi-region write path without tolerating the latency of synchronous cross-region consistency, we need to design a solution that ensures data availability and eventual consistency across multiple regions while minimizing propagation delays. The key is to leverage asynchronous replication and carefully managed event-driven architectures.

#### Triage

The question is asking for a strategy to handle writes in a distributed system where synchronous cross-region consistency is not feasible due to latency constraints. This involves understanding the trade-offs between immediate availability, eventual consistency, and data redundancy. Assumptions include that the application can tolerate some level of inconsistency until updates are propagated across regions.

#### First-Principles Solution
Identify Write Regions: Designate one or more write regions where all writes will originate.
Asynchronous Replication: Ensure that after a write is committed in the write region, it is asynchronously replicated to other read regions.
Eventual Consistency: Implement mechanisms to ensure that reads from different regions may return stale data until eventually consistent updates are propagated.

#### Implementation
Steps
Primary Write Region:
Choose one or more primary regions (e.g., closest to the user base) as write regions.
Replication Strategy:
Use a distributed database system that supports eventual consistency, such as Cassandra or DynamoDB with multi-active replication.
Eventual Consistency Mechanisms:
Implement a mechanism like Apache Pulsar or Kafka for handling asynchronous messages between regions.
Read Regions:
Designate other regions as read-only to avoid the complexity of synchronizing reads.
Health Checks and Failover:
Set up health checks in each region to detect failures and trigger failover if necessary.

#### Example
Solution

Let's use AWS as an example, leveraging DynamoDB for its strong eventual consistency model:

DynamoDB Write Region Setup:
BASH
# Define a primary write region (e.g., US-East-1)
Asynchronous Replication:
Use DynamoDB Streams to capture changes and push them to an S3 bucket.
Use AWS Lambda functions to read from S3 and replicate data to secondary regions.
User Writes
DynamoDB Write Region(US-East-1)
S3 Bucket
Lambda Function forReplication
DynamoDB Read Regions (e.g.,US-West-2, EU-West-1)

#### Closing Summary

The strategy involves designing a multi-region architecture where writes are initially committed to the primary region and then asynchronously replicated to other regions. This approach ensures that the system can handle high availability while managing eventual consistency.

#### Common Mistakes/Follow-Ups
Not Handling Failovers: Ensure there are mechanisms in place for failover if the write region becomes unavailable.
Latency Concerns with Synchronous Replication: Always use asynchronous replication to avoid latency issues.
Data Consistency Across Regions: Use tools like DynamoDB Streams and AWS Lambda to manage eventual consistency.

**Comparison:**

| Region | Role       | Operations   |
|--------|------------|--------------|
| US-East-1 | Write     | Writes, S3 Bucket Updates        |
| US-West-2 | Read      | Reads from DynamoDB             |
| EU-West-1 | Read      | Reads from DynamoDB             |

**Code:**

```python
# Example Python code for handling writes in the primary region using Boto3
import boto3

def write_to_dynamodb(item):
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('MyTable')
    response = table.put_item(Item=item)
    return response

item = {'id': '1', 'name': 'John Doe'}
write_response = write_to_dynamodb(item)
print(write_response)
```

**Example input / output:**

```

Input:

PYTHON
Copy
item = {'id': '1', 'name': 'John Doe'}
write_response = write_to_dynamodb(item)
print(write_response)

Output:

JSON
Copy
{
  "ResponseMetadata": {
    "RequestId": "A1B2C3D4E5F6G7H8I9J0",
    "HTTPStatusCode": 200,
    "HTTPHeaders": {
      ...
    },
    "RetryAttempts": 0
  }
}
```

**Data:**

```json
{
  "Item": {
    "id": "1",
    "name": "John Doe"
  }
}
```

**Flow:**

1. User writes
2. DynamoDB Write Region(US-East-1)
3. S3 Bucket
4. Lambda Function forReplication
5. DynamoDB Read Regions (e.g.,US-West-2, EU-West-1)

