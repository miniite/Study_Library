---
type: "practice"
difficulty: "Deep"
---

# What's your view on serving master data via an API/GraphQL 'entity service' vs publishing to a shared data product in the lakehouse — when is each the right consumption pattern?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ MDM for Batch and Streaming](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/MDM_for_Batch_and_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To serve master data, we often face two primary consumption patterns: delivering it via an API or GraphQL 'entity service' versus publishing to a shared data product in a lakehouse. Each approach has its own merits and trade-offs, which I'll explore from first principles.

The core question here is about understanding the differences between these two approaches and when each might be more suitable based on factors like performance, consistency, security, and operational complexity. Let's start with some assumptions: we're dealing with a large-scale enterprise environment where data quality, availability, and governance are critical, and we have multiple consumers across various applications.

#### Triage

The question is probing your understanding of the different consumption patterns for master data in an enterprise context. You need to analyze whether delivering master data via an API or GraphQL is better suited than publishing it directly into a shared lakehouse dataset. Key factors include performance, scalability, consistency, security, and ease of integration.

#### First-Principles Solution
Serving Master Data via API/GraphQL 'Entity Service'

This approach involves exposing the master data through APIs or GraphQL endpoints that clients can query. The benefits are:

Granularity: You can control which fields and entities each client gets access to, ensuring only necessary data is sent.
Security: Fine-grained permissions can be enforced at the API level, enhancing security by limiting who can see what data.
Performance: Optimizations like caching, rate limiting, and partial responses can be implemented more easily with APIs.

However, this approach may introduce complexity in terms of maintaining and scaling these services. You need to ensure that your MDM system is robust enough to handle multiple concurrent requests efficiently.

Publishing Master Data to a Shared Data Product

This involves storing the master data directly into a shared data product like a lakehouse and allowing consumers to access it via standard queries (e.g., SQL, Apache Spark). The benefits are:

Simplicity: Consumers can use familiar query languages to interact with the data, reducing the learning curve.
Performance: Optimized for large-scale queries and analytics tasks, which might be challenging with APIs.

#### Cons
istency: Easier to maintain a single source of truth by updating the lakehouse directly.

The drawbacks include:

Security: More difficult to enforce fine-grained access control over shared data products.
Scalability: Potentially less flexible in terms of serving different types and volumes of queries efficiently.
Operational Complexity: Managing a shared dataset requires careful planning and may introduce additional overhead.
When is Each Approach Right?

API/GraphQL 'Entity Service':

Suitable when data access needs to be highly controlled, with granular permissions required.
Ideal for scenarios where the same master data might need to be accessed in different ways by various applications, possibly requiring different subsets of the data.
Best fit for real-time or near-real-time data serving requirements.

Shared Data Product:

Appropriate when multiple consumers need access to a consistent view of the data and can handle the shared nature of the dataset.
Preferable in batch processing environments where complex analytics are performed on the master data.
Works well for scenarios where performance, consistency, and ease of use outweigh security concerns.

#### Closing Summary

In summary, both approaches have their strengths and weaknesses. The choice depends heavily on the specific requirements of your enterprise environment, including the type of access needed, the complexity of the queries, and the security needs. A balanced approach often involves combining elements of both patterns, such as using APIs for real-time updates while leveraging a shared data product for bulk analytics.

#### Common Mistakes/Follow-Ups

#### Common mistakes
include over-abstracting the data model through APIs without considering performance implications or underestimating the importance of consistency in a shared data environment. Follow-ups might involve discussing specific use cases, how to implement security measures effectively with APIs, and strategies for optimizing both approaches for high throughput.

**Comparison:**

```
Use Case	API/GraphQL 'Entity Service'	Shared Data Product
Real-time Updates	✓	-
Fine-grained Access Control	✓	-
Batch Analytics	-	✓
Simplicity for Consumers	-	✓
```

**Code:**

```python
# Example API endpoint definition (using Flask)
from flask import Flask, request

app = Flask(__name__)

@app.route('/master-data', methods=['GET'])
def get_master_data():
    # Implement logic to retrieve and return master data based on query parameters
    pass
```

**Example input / output:**

```text
# Sample input request to API/GraphQL service
{
  "query": "{ masterData(id: 1) { name, description } }"
}

# Sample output response from API/GraphQL service
{
  "data": {
    "masterData": {
      "name": "Customer A",
      "description": "Description of Customer A's master record"
    }
  }
}
```

**Data:**

```yaml
# Example schema for a shared lakehouse dataset
master-data:
  id: integer
  name: string
  description: string
  created_at: datetime
  updated_at: datetime
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. API/GraphQL Service
6. Is the request in real-time?
7. Do we need fine-grainedaccess control?
8. Return data via API
9. Write to shared lakehouse
10. Read from shared lakehouse

