---
type: "practice"
difficulty: "Basic"
---

# What is Azure Cosmos DB at a high level and when would you reach for it?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Azure SQL and App Service Deployments](<../../../../TOOLS/Azure/Azure_Platform/Azure_SQL_and_App_Service_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Azure Cosmos DB is a globally distributed, multi-model database service that allows developers to build scalable applications with ease. At a high level, it offers automatic scalability across multiple regions, ensuring low latency and high availability. You would reach for Azure Cosmos DB when you need a highly available, performant, and scalable database solution without the complexity of managing infrastructure.

#### Triage
What is Azure Cosmos DB?

Azure Cosmos DB is part of Microsoft's broader cloud ecosystem and is designed to handle various types of data and workloads efficiently. The core questions here are:

What makes it unique?
When would you use it compared to other database solutions like SQL Server or MongoDB?

#### First-Principles Solution
Understanding Azure Cosmos DB
1. Global Distribution

Azure Cosmos DB automatically replicates data across multiple regions, ensuring low latency and high availability. This is crucial for applications with global users.

2. Multi-Model Support

It supports several database models including SQL, MongoDB, Cassandra, Gremlin (for graph databases), and Table API, making it highly versatile. You can choose the model that best fits your application's needs without changing providers.

3. Automatic Scalability

The service automatically scales throughput and storage capacity based on real-time demand, eliminating the need for manual scaling operations.

4. Consistency Models

Azure Cosmos DB supports multiple consistency levels, allowing you to balance between strong and eventual consistency depending on your application requirements.

Common Use Cases
Global Applications: When building applications with users across different geographical regions.
High Availability: For critical applications that require minimal downtime.
Scalable Workloads: For applications where data volume or user traffic can spike unpredictably.
Multi-Model Support: When your application requires multiple types of data models, such as relational and document-based.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Azure Cosmos DB is a powerful tool for building modern, global-scale applications. However, it comes with its own trade-offs:

Cost: Being a managed service, Azure Cosmos DB can be more expensive compared to self-managed solutions.
Complexity: While the service abstracts much of the complexity, understanding consistency models and choosing the right model can still require effort.

#### Follow-Ups
How does it handle data backups?
Can you share an example scenario where Azure Cosmos DB outperformed another database solution?

**Comparison:**

| Operation | Description |
|-----------|-------------|
| Create Database | Azure Cosmos DB creates a new database with the specified name. |
| Create Container | Within a database, a container is created to manage data. |
| Upsert Item | Inserts or updates an item in the container with given partition key. |

**Code:**

```python
# Example Python code to create a Cosmos DB client (simplified)
from azure.cosmos import CosmosClient, PartitionKey

client = CosmosClient(url="https://your-cosmosdb-url", credential="your-credential")
database_name = "sample-database"
container_name = "sample-container"

database = client.create_database_if_not_exists(id=database_name)
container = database.create_container(
    id=container_name,
    partition_key=PartitionKey(path="/partitionKey"),
    offer_throughput=400
)

# Insert sample data
item = {
    "id": "1",
    "name": "Sample Item"
}
response = container.upsert_item(body=item)
```

**Example input / output:**

```text
Input:
- Database name: sample-database
- Container name: sample-container
- Partition key path: /partitionKey
- Offer throughput: 400

Output:
- Created database: sample-database
- Created container: sample-container with partition key on '/partitionKey'
- Inserted item successfully
```

**Data:**

```json
{
  "databaseName": "sample-database",
  "containerName": "sample-container",
  "partitionKeyPath": "/partitionKey",
  "offerThroughput": 400,
  "items": [
    {
      "id": "1",
      "name": "Sample Item"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is it a global application?
7. Use Azure Cosmos DB for itsglobal distribution capabilities
8. Is there a need for highavailability and scalability?
9. Consider traditional databaseslike SQL Server or MongoDB

