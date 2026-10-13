---
type: "practice"
difficulty: "Expert"
---

# How would you design multi-region write handling for a fintech ledger where you cannot tolerate lost or double-applied transactions?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Read Your Writes with Replicas](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design multi-region write handling for a fintech ledger that ensures no lost or double-applied transactions, we need to focus on ensuring consistency and reliability across multiple regions while minimizing latency and downtime. At an expert level, this involves leveraging cloud services' built-in features like distributed databases, consensus mechanisms, and eventual consistency strategies.

#### Triage
Understanding the Question

The question is probing your understanding of distributed systems, particularly in designing a highly available and consistent ledger system that meets strict requirements for transactions. Key aspects to consider include:

#### Cons
istency: Ensuring no lost or double-applied transactions.
Availability: Making sure the system remains functional across multiple regions.
Latency: Minimizing the time between when a transaction is initiated and when it's processed.

#### Assumptions
The ledger system involves multiple regions (e.g., North America, Europe).
Transactions must be consistent across all regions to avoid any discrepancies.
Lost or double-applied transactions are not tolerable in this context.
Step-by-Step Solution
1. Choose the Right Consistency Model

For a fintech ledger that cannot tolerate lost or double-applied transactions, we need to use a strong consistency model such as Strongly Consistent (e.g., using Paxos or Raft). However, achieving strong consistency across multiple regions is challenging due to network latency and partition tolerance. Therefore, we can opt for a Quasi-Local Strong Consistency approach.

2. Use a Distributed Database

A distributed database like Amazon DynamoDB or Google Cloud Spanner can be used. These databases support strong consistency within a region but may require additional steps to achieve cross-region consistency.

3. Implement Eventual Consistency with Staged Writes

To handle writes across multiple regions, we can implement a staged write approach where:

Primary Write: The first transaction is written to the primary region.
Replication: The transaction is then replicated asynchronously to other regions using techniques like AWS DDB Streams or Google Cloud Pub/Sub.
4. Handle Conflicts with Conflict Resolution

In case of conflicts (e.g., a write from another region arrives before the replication), we need a conflict resolution strategy:

Monotonic Clocks: Ensure that each transaction has a unique timestamp to detect and resolve conflicts.
Conflict-Free Replicated Data Types (CRDTs): Use CRDTs for state consistency, which can merge conflicting updates automatically.
5. Leverage Cloud Services

Leverage cloud services like AWS Global Tables or Google Cloud Spanner’s multi-region support:

AWS Global Tables: Provides global transactional consistency across regions.
Google Cloud Spanner: Supports strong consistency and allows for reads from any region while writes are managed globally.
6. Implement Cross-Region Conflict Resolution

Set up a system to detect and resolve conflicts between regions in real-time or batch processing:

Conflict Detection: Use a distributed hash table (DHT) or a similar mechanism to track conflicts.
Resolution Mechanism: Develop a robust resolution logic that could involve manual intervention or automated conflict resolution.
7. Monitor and Audit

Continuously monitor the system for any issues and set up comprehensive auditing mechanisms to detect and rectify inconsistencies early:

Monitoring Tools: Use cloud-native monitoring tools like AWS CloudWatch, Google Cloud Operations.
Audit Logs: Maintain detailed logs of all transactions for forensic analysis if needed.

#### Closing Summary

To ensure a multi-region write handling system in a fintech ledger that avoids lost or double-applied transactions, we need to employ strong consistency models, distributed databases, and conflict resolution strategies. By leveraging cloud services’ built-in features and implementing robust monitoring, we can achieve high availability and consistency while minimizing risks.

#### Common Mistakes
and Follow-Ups
Ignoring Latency: Failing to account for network latency can lead to incorrect transaction handling.
Over-reliance on Asynchronous Replication: Not having a fallback plan or conflict resolution strategy can result in data inconsistencies.
Lack of Monitoring and Auditing: Without proper monitoring, issues may go unnoticed until they become critical.

**Comparison:**

| Region   | Status       |
|----------|--------------|
| us-east-1| Success      |
| eu-west-1| Conflict Resolved|
| ap-southeast-1| Successful Write|

**Code:**

```python
# Example of using AWS Global Tables with Python Boto3

import boto3

dynamodb = boto3.resource('dynamodb')

def write_transaction(transaction_data):
    global_table = dynamodb.Table('GlobalTable')

    response = global_table.put_item(Item=transaction_data)
    return response
```

**Example input / output:**

```text
Input:
Transaction Data: {
  "region": "us-east-1",
  "transaction_id": "1234567890",
  "amount": 100.0,
  "timestamp": "2023-10-01T12:00:00Z"
}

Output:
{
  "region": "us-east-1",
  "transaction_id": "1234567890",
  "amount": 100.0,
  "timestamp": "2023-10-01T12:00:00Z",
  "status": "success"
}
```

**Data:**

```json
{
  "region": "us-east-1",
  "transaction_id": "1234567890",
  "amount": 100.0,
  "timestamp": "2023-10-01T12:00:00Z"
}
```

**Flow:**

1. Yes
2. No
3. Transaction Initiated
4. Is Local Region?
5. Write to Primary Region
6. Replicate to Other Regions
7. Conflict Detection Mechanism
8. Resolve Conflict(Manual/Automated)
9. Monitor and Audit System

