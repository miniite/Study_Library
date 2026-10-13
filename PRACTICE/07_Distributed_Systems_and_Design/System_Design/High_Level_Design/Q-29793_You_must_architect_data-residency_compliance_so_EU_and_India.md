---
type: "practice"
difficulty: "Expert"
---

# You must architect data-residency compliance so EU and India user data never leaves-region while a global feed still works — how do you split storage, keep joins possible, and prove compliance to auditors?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Audit and Compliance Platforms](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Audit_and_Compliance_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of ensuring that EU and Indian user data never leave their regions while maintaining a global feed, we need to design a solution that splits storage, retains join capabilities, and provides verifiable compliance to auditors. This involves leveraging region-specific databases for local storage, implementing cross-region synchronization for the global feed, and using cryptographic techniques for data protection.

Firstly, the core concept is to use multi-region database technologies that support data residency while allowing for seamless data access and synchronization across regions. For this solution, we'll utilize a combination of AWS DynamoDB Global Tables or Google Cloud Spanner, which inherently support cross-region replication but with strict data residency controls.

#### Triaging the Question

The question asks us to design a high-level architecture (HLD) that ensures EU and Indian user data remain within their respective regions while still maintaining a global feed. This involves several key aspects:

Data Residency Compliance: Ensuring that specific regions handle sensitive data locally.
Global Feed: Maintaining a unified view of the data for all users, regardless of location.
Join Operations: Retaining the ability to join and query across different databases without compromising data residency.

#### Assumptions
The system must support real-time data processing and querying.
Compliance with GDPR and other regional regulations is critical.
Data integrity and security are paramount.

#### First-Principles Solution
1. Multi-Region Database Setup

We will use AWS DynamoDB Global Tables or Google Cloud Spanner to create separate databases for EU, India, and a global hub. Each region-specific database will store the corresponding data, ensuring that sensitive information remains within its designated jurisdiction.

EU Data
DynamoDB
India Data
DynamoDB
Global Hub Data
DynamoDB
2. Cross-Region Synchronization

To maintain a global feed, we will implement cross-region synchronization between the EU/India-specific databases and the global hub. This can be achieved using AWS Global Tables or Google Cloud Spanner’s built-in replication capabilities.

EU Data
DynamoDB
India Data
DynamoDB
Global Hub Data
DynamoDB
Global Hub Data Replication
Global Hub Data Replication
3. Data Access and Join Operations

For join operations, we will use techniques like sharding or distributed joins to ensure that data remains in its respective region while still allowing for cross-region queries. This can be implemented using tools like Apache Flink or Spark Streaming.

EU Data
DynamoDB
India Data
DynamoDB
Global Hub Data
DynamoDB
Global Hub Data Replication
Global Hub Data Replication
Sharded Global Join
Unified Query
4. Verification and Compliance

To prove compliance to auditors, we will implement logging and monitoring systems that track data access and replication activities. We can use AWS CloudTrail or Google Cloud Audit Logs to generate detailed logs and alerts.

EU Data
DynamoDB
India Data
DynamoDB
Global Hub Data
DynamoDB
Global Hub Data Replication
Global Hub Data Replication
Sharded Global Join
Unified Query
Auditor Compliance Logs
CloudTrail/CloudAuditLogs

#### Closing Summary

The architecture involves setting up separate databases for EU and India while maintaining a global hub. Cross-region synchronization ensures that the global feed is updated in real-time, and distributed join operations allow for seamless data access without compromising regional compliance. Logging and monitoring systems provide verifiable proof of compliance to auditors.

#### Common Mistakes/Follow-Ups
Not Considering Edge Cases: Ensure to handle scenarios where network latency or failures might disrupt synchronization.
Ignoring Security Best Practices: Implement strong encryption, access controls, and regular security audits.

#### Performance Considerations
Optimize queries and data sharding strategies to ensure low-latency performance.

**Comparison:**

| Database | Regions | Key Schema | Join Operations Support |
|----------|---------|------------|------------------------|
| EUUserData | Europe   | id, email  | Yes                    |
| IndiaUserData | Asia     | id, email  | Yes                    |
| GlobalHubData | World    | id, email  | Yes                    |

**Code:**

```python
# Example Python code for setting up DynamoDB Global Table using boto3
import boto3

dynamodb = boto3.resource('dynamodb')
global_table = dynamodb.create_global_table(
    ReplicationGroup=[
        {'RegionName': 'eu-west-1'},
        {'RegionName': 'ap-south-1'}
    ]
)
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Insert EU user data into `EUUserData` table. | Data replicated to GlobalHubData and stored in EUUserData. |
| Query global feed for a specific user from India. | Joins `IndiaUserData`, `GlobalHubData`, and returns result from IndiaUserData. |

**Data:**

```json
{
  "EUData": {
    "TableName": "EUUserData",
    "KeySchema": [
      {"AttributeName": "id", "KeyType": "HASH"}
    ],
    "GlobalSecondaryIndexes": [
      {
        "IndexName": "UserEmailIndex",
        "KeySchema": [
          {"AttributeName": "email", "KeyType": "RANGE"}
        ],
        "Projection": {"ProjectionType": "ALL"}
      }
    ]
  },
  "IndiaData": {
    "TableName": "IndiaUserData",
    "KeySchema": [
      {"AttributeName": "id", "KeyType": "HASH"}
    ],
    "GlobalSecondaryIndexes": [
      {
        "IndexName": "UserEmailIndex",
        "KeySchema": [
          {"AttributeName": "email", "KeyType": "RANGE"}
        ],
        "Projection": {"ProjectionType": "ALL"}
      }
    ]
  },
  "GlobalHubData": {
    "TableName": "GlobalHubData",
    "KeySchema": [
      {"AttributeName": "id", "KeyType": "HASH"}
    ],
    "GlobalSecondaryIndexes": [
      {
        "IndexName": "UserEmailIndex",
        "KeySchema": [
          {"AttributeName": "email", "KeyType": "RANGE"}
        ],
        "Projection": {"ProjectionType": "ALL"}
      }
    ]
  }
}
```

**Flow:**

1. EU User Data
2. DynamoDB EU
3. India User Data
4. DynamoDB India
5. Global Feed Query
6. DynamoDB Global Hub
7. Join Operations
8. Unified Response
9. Audit Logs
10. CloudTrail/CloudAuditLogs

