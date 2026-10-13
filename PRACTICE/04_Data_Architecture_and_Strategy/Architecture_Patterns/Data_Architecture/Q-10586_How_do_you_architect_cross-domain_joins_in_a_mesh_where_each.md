---
type: "practice"
difficulty: "Expert"
---

# How do you architect cross-domain joins in a mesh where each domain owns its data product? What's the performance and governance cost of federated query vs replicating data products, and when do you choose each?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-domain Joins in a Mesh](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Cross-domain_Joins_in_a_Mesh.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a highly distributed microservices architecture, cross-domain joins can become complex due to the autonomy and ownership of data by different domains. To effectively architect these joins in a "mesh" environment, we need to consider multiple factors including performance, governance, and cost implications. In this answer, I will break down the challenges involved, explore federated query vs. data replication, and discuss when each approach is more suitable.

#### Triaging the Question

The key aspects of this question revolve around understanding how different domains handle their own data while still allowing for necessary cross-domain joins. The primary questions being asked are:

How can we implement cross-domain joins in a mesh architecture?
What are the trade-offs between federated querying and replicating data across domains?
When should each approach be chosen?
Key Assumptions
Each domain owns its own data products, meaning no single domain has access to all other domains' data.
The system is designed as a microservices mesh with APIs for inter-domain communication.

#### First Principles Solution

To implement cross-domain joins in such an architecture, we need to balance performance and governance costs. Here’s how:

Federated Query Approach:

Definition: Federated querying involves writing queries that span multiple domains without replicating the data.

#### Performance Considerations
While federated querying can be highly performant by leveraging distributed query execution engines, it introduces additional network latency and potential reliability issues due to inter-domain communication.
Governance and Complexity: This approach requires well-defined APIs and strict governance policies to ensure that queries are optimized and do not violate domain boundaries.

Data Replication Approach:

Definition: Data replication involves replicating data from one domain into another, allowing for local joins.

#### Performance Considerations
Local joins can be more efficient as they avoid network latency and reduce the load on distributed query engines.
Governance and Complexity: This approach requires careful management to ensure consistency between domains and to handle potential conflicts.
Performance and Governance Cost Analysis
Federated Query
Benefits:
No need for data duplication, reducing storage costs.
Potentially lower governance overhead as fewer copies of data exist.
Challenges:
Increased network latency due to inter-domain queries.
Potential reliability issues if domain systems are not highly available.
Data Replication
Benefits:
Local joins can be faster and more efficient.
Simplified query execution without inter-domain communication.
Challenges:
Increased storage costs for duplicated data.
Higher governance complexity to ensure data consistency across domains.
When to Choose Each Approach
Federated Query:
Use this approach when the performance benefits of local joins are minimal or non-existent, and minimizing data duplication is critical.
Suitable for highly distributed systems with strict latency requirements or limited network bandwidth.
Data Replication:
Opt for this approach in scenarios where performance is a high priority, and local join efficiency outweighs the costs of data duplication.
Appropriate when dealing with large datasets that require frequent access patterns.

#### Closing Summary

In summary, architecting cross-domain joins in a mesh requires careful consideration of both technical and organizational factors. Federated querying offers lower governance complexity but may introduce performance overhead due to network latency, while replicating data provides better local join performance at the cost of increased storage and potential governance challenges.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the trade-offs between performance and governance when choosing an approach.

#### Follow-Up
Discussing real-world examples or analogies to further illustrate these points. For example, comparing a federated query system to a database federation in financial systems versus replicating customer data across regions in e-commerce.

**Comparison:**

```
Domain	Data Products	Join Strategy
1	Sales, Inventory	Federated Query (latency sensitive)
2	Customers, Orders	Replication (frequent access)
```

**Code:**

```
MERMAID
Copy
graph TB;
    A[Domain 1] -->|Query| B[Domain 2];
    B -->|Data| C[Database Federation Service];
    C -->|Join| D[Result Set];
```

**Example input / output:**

```text
**Example Input**: A query to join `Sales` from Domain 1 with `Orders` from Domain 2.
- **Federated Query Example**:
  - Request: Query for sales and orders data concurrently.
  - Response: Data fetched from both domains, joined locally at the service layer.

- **Data Replication Example**:
  - Request: Join local copies of sales and orders data.
  - Response: Local join executed efficiently, reducing network latency.
```

**Data:**

```json
{
  "domains": [
    {
      "name": "Domain 1",
      "dataProducts": ["Sales", "Inventory"]
    },
    {
      "name": "Domain 2",
      "dataProducts": ["Customers", "Orders"]
    }
  ]
}
```

**Flow:**

1. Domain Request
2. Federated Query Service
3. Query Execution AcrossDomains
4. Joined Data Returned

