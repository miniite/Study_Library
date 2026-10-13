---
type: "practice"
difficulty: "Expert"
---

# How does DynamoDB's Global Secondary Index differ from a relational index in cost and consistency?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Random Keys and Index Locality](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Random_Keys_and_Index_Locality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

DynamoDB’s Global Secondary Index (GSI) offers flexibility and performance benefits compared to traditional relational database indexes, but comes with specific trade-offs related to cost and consistency. Let's break down how GSIs differ from relational indexes in these aspects.

#### Triage

The question is probing your understanding of the key differences between DynamoDB’s GSI and relational database indexes regarding cost and consistency. It also asks you to compare their practical implications.

#### Assumptions
We assume a basic understanding of both DynamoDB and traditional relational databases.
The focus will be on cost and consistency, with other factors like query performance being mentioned but not the primary focus.

#### First-Principles Solution
Relational Indexes

Relational indexes (B-trees) are designed to provide fast access to records based on a specific column. They are stored alongside the table data and offer strong consistency models, such as ACID compliance in systems like PostgreSQL or MySQL. However, they come with increased storage costs due to additional index data.

DynamoDB Global Secondary Indexes

DynamoDB GSIs are designed for NoSQL databases and offer more flexibility. A GSI allows you to define a secondary key on any attribute of the primary key, enabling efficient querying based on that attribute. GSIs are stored in a separate table from the base table, which impacts both cost and consistency.

Cost
Relational Indexes: The storage cost is directly proportional to the size of the index, which can be significant for large tables.
DynamoDB GSIs: GSIs incur additional read/write capacity units (RCUs/WCUs) as they need to support queries. However, unlike relational databases, DynamoDB offers provisioned and on-demand billing options, making cost more predictable or variable depending on your needs.

#### Cons
istency
Relational Indexes: Typically offer strong consistency guarantees, where updates are immediately visible in all replicas.
DynamoDB GSIs: By default, GSI queries can return eventually consistent results. However, you can choose strongly consistent reads for a higher cost and lower performance.

#### Closing Summary
+ Common Mistakes/Follow-Ups

When choosing between relational indexes and DynamoDB GSIs, consider your application's consistency requirements and budget constraints. GSIs offer more flexibility and better scalability but may introduce additional costs due to read/write capacity units. Understanding these trade-offs is crucial for designing efficient and cost-effective database architectures.

#### Common mistakes
include not properly understanding the consistency models or underestimating the cost implications of using GSIs.

#### Common Mistakes
Assuming GSI Consistency: Many developers assume that GSI queries will always be consistent, leading to performance issues.
Ignoring Cost Implications: Failing to account for RCU/WCU costs can lead to unexpected expenses.
Overdesigning Indexes: Creating too many GSIs or overly complex query patterns without optimizing for read/write performance.

#### Follow-Ups
How do you manage the cost and consistency trade-offs in a production environment?
Can you provide an example where GSI would be more beneficial than a relational index, and vice versa?

**Comparison:**

| Attribute | Description             | Relational Indexes       | DynamoDB GSIs        |
|-----------|-------------------------|-------------------------|---------------------|
| Storage   | Additional index storage | + for large tables      | Separate table,     |
|          |                         |                         | no extra storage    |
|           |                         |                         | overhead            |
| Cost      |                          | High                    | RCU/WCU costs       |
| Consistency |                        | Strong                  | Eventually         |
|           |                         | (ACID)                  | consistent,         |
|           |                         |                         | strongly consistent |

**Code:**

```python
# Example of defining a GSI in DynamoDB using Boto3

import boto3

dynamodb = boto3.resource('dynamodb')

table = dynamodb.Table('exampleTable')
gsi = table.create_global_secondary_index(
    IndexName='exampleGSI',
    KeySchema=[
        {
            'AttributeName': 'email',  # The key attribute for the GSI
            'KeyType': 'HASH'          # Partition key
        }
    ],
    Projection={
        'ProjectionType': 'ALL'
    },
    ProvisionedThroughput={
        'ReadCapacityUnits': 5,
        'WriteCapacityUnits': 5
    }
)
```

**Example input / output:**

```text
Input:
- Create a GSI on the 'email' attribute of an existing DynamoDB table.

Output:
- DynamoDB will create a new global secondary index with specified throughput settings.
- The base table and the new GSI will be operational, allowing for efficient querying based on email addresses.
```

**Data:**

```json
{
  "table": {
    "AttributeDefinitions": [
      {"AttributeName": "id", "AttributeType": "S"},
      {"AttributeName": "email", "AttributeType": "S"}
    ],
    "KeySchema": [
      {"AttributeName": "id", "KeyType": "HASH"},
      {"AttributeName": "email", "KeyType": "RANGE"}
    ]
  },
  "gsi": {
    "IndexName": "exampleGSI",
    "ProvisionedThroughput": {
      "ReadCapacityUnits": 5,
      "WriteCapacityUnits": 5
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Application's cost is a keyfactor
6. Start
7. Is the application's consistencyrequirement high?
8. Use relational indexes
9. Do you need flexibility inquerying data?
10. Use DynamoDB GSIs
11. Use simple primary keys
12. Consider DynamoDBon-demand pricing model
13. Monitor and optimizeRCU/WCU usage

