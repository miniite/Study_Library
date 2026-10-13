---
type: "practice"
difficulty: "Deep"
---

# What's the difference between DynamoDB's Query and Scan cost model, and why does adding a FilterExpression not reduce consumed capacity? How does this change your data model?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ DynamoDB Modeling and Capacity](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer the question about the differences in cost models between DynamoDB’s Query and Scan, as well as why a FilterExpression doesn't reduce consumed capacity, let's start with an overview of each concept and then delve into the details.

In summary:

DynamoDB's Query operation allows for efficient filtering based on a partition key and a sort key range.
DynamoDB's Scan operation is used to read every item in a table or a secondary index, making it less performant due to high consumed capacity and cost.
Adding a FilterExpression does not reduce consumed capacity because FilterExpression only filters the data after the items have been retrieved from storage, and thus, the number of items retrieved is still counted towards consumed capacity.

Now let’s delve into these concepts in more detail:

#### Triage
Understanding the Question

The question aims to explore:

Differences in Cost Models: We need to understand how Query and Scan operations differ in terms of their cost models.
Impact of FilterExpression: We must explain why a FilterExpression, which allows for filtering data, does not impact the consumed capacity.
Step-by-Step Solution: From First Principles
1. Understanding Query Operation

DynamoDB's Query operation is designed to fetch items from a table or index based on their partition key and (optionally) sort key ranges. This operation has several advantages:

Efficiency: It operates on the secondary index, where it can leverage its sorted nature.
Low Consumed Capacity: The consumed capacity of Query depends only on the number of items with a particular partition key prefix.
2. Understanding Scan Operation

In contrast, DynamoDB's Scan operation reads every item in a table or index:

High Consumed Capacity: Regardless of how many items match the filter condition, the entire table is scanned.
Costly: This operation can be expensive due to its high consumed capacity and cost.
3. FilterExpression and Consumed Capacity

A FilterExpression allows you to specify additional filtering criteria after the initial retrieval of items from storage:

Post-processing Filtering: The FilterExpression is applied on the retrieved items.

#### Cons
umed Capacity Unchanged: Since the items are still retrieved based on the Query or Scan, the consumed capacity remains the same. FilterExpression only filters out data that doesn’t match the condition, but it does not reduce the number of items read.
4. Data Model Considerations

Given these differences, your data model should be designed with appropriate considerations:

Use Query for Specific Retrievals: Utilize Query when you need to retrieve specific sets of data based on partition key and sort key ranges.
Minimize Scans: Use Scan sparingly due to its high cost. Prefer Query where possible to reduce the number of items read.

#### Closing Summary
+ Common Mistakes/Follow-ups

In conclusion, DynamoDB's Query is more efficient than Scan, with lower consumed capacity and cost for specific data retrieval needs. However, a FilterExpression does not reduce consumed capacity because it filters post-retrieval. This distinction impacts the design of your data model, emphasizing the importance of using Query when possible to optimize performance.

#### Common mistakes
include:

Overusing Scan, leading to high costs and degraded performance.
Ignoring the implications of FilterExpression on consumed capacity.

#### Follow-up
questions might explore scenarios where a hybrid approach combining both Query and Scan could be beneficial, or how to design more efficient data models for large-scale applications.

**Comparison:**

| Operation | Cost Model                  | Consumed Capacity |
|-----------|-----------------------------|------------------|
| Query     | Partitions + Sort Key Range  | Low              |
| Scan      | Entire Table or Index       | High             |

**Code:**

```python
# Example Python code using Boto3 to illustrate DynamoDB operations

import boto3

dynamodb = boto3.resource('dynamodb')

table = dynamodb.Table('MyTable')

# Query example
response_query = table.query(
    KeyConditionExpression=boto3.dynamodb.conditions.Key('partition_key').eq('key_value')
)

# Scan example with FilterExpression
response_scan = table.scan(
    FilterExpression=boto3.dynamodb.conditions.Attr('attribute_name').gt(10)
)
```

**Example input / output:**

```text
# Query example input-output

Input:
response = table.query(
    KeyConditionExpression=boto3.dynamodb.conditions.Key('partition_key').eq('key_value')
)

Output:
{
  "Items": [
    {"sort_key": "value1", "attribute_name": "data1"},
    {"sort_key": "value2", "attribute_name": "data2"}
  ]
}

# Scan example input-output

Input:
response = table.scan(
    FilterExpression=boto3.dynamodb.conditions.Attr('attribute_name').gt(10)
)

Output:
{
  "Items": [
    {"partition_key": "key_value1", "sort_key": "value1", "attribute_name": "data12"},
    {"partition_key": "key_value2", "sort_key": "value2", "attribute_name": "data34"}
  ]
}
```

**Data:**

```json
{
  "Table": {
    "AttributeDefinitions": [
      {"AttributeName": "partition_key", "AttributeType": "S"},
      {"AttributeName": "sort_key", "AttributeType": "N"}
    ],
    "KeySchema": [
      {"AttributeName": "partition_key", "KeyType": "HASH"},
      {"AttributeName": "sort_key", "KeyType": "RANGE"}
    ]
  },
  "SecondaryIndexes": [
    {
      "IndexName": "sort_key_index",
      "Projection": {"ProjectedAttributes": ["attribute_name"]},
      "KeySchema": [{"AttributeName": "partition_key", "KeyType": "HASH"}, {"AttributeName": "sort_key", "KeyType": "RANGE"}]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Determine if data can befiltered by partition key
5. Use Query operation
6. Consider using Scan withFilterExpression
7. Apply Query and filter on theretrieved items
8. Scan entire table/index, applyFilterExpression post-retrieval
9. Low consumed capacity
10. High consumed capacityregardless of filtering

