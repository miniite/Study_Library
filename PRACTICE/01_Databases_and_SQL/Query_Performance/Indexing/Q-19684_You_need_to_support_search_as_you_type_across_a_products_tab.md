---
type: "practice"
difficulty: "Deep"
---

# You need to support 'search as you type' across a products table. Compare a trigram index, a prefix B-tree, and delegating to a search service in terms of latency, freshness, and ops cost.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Indexing Read State at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Indexing_Read_State_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the "search as you type" requirement across a products table, we need to compare three indexing strategies: trigram index, prefix B-tree, and delegating to a search service. The evaluation criteria are latency, freshness, and operations cost. Let's break down each approach in detail.

#### Triage

The question asks us to evaluate three different indexing and query optimization techniques for the "search as you type" functionality. We need to consider how well each method performs in terms of response time (latency), ability to provide up-to-date results (freshness), and operational overhead (cost).

Trigram Index

A trigram index breaks down text into three-character sequences, which allows for approximate string matching. This is particularly useful for fuzzy searches like "search as you type" where users might be typing partial words.

Latency:

Trigram indexes can perform well in terms of latency because they are typically cached and optimized for quick lookups.
However, trigrams may introduce some overhead due to the need to match multiple sequences.

Freshness:

The freshness depends on how often the index is updated. If updates are frequent and managed properly, it can maintain good freshness.
Trigram indexes might require periodic rebuilding or incremental updates, which could impact performance during these operations.

Ops Cost:

Trigram indexes generally have a moderate ops cost due to their need for indexing and potential cache management.
However, they do not require complex infrastructure like full-text search services.
Prefix B-Tree

A prefix B-tree is an index that stores prefixes of words in nodes. This can help in achieving efficient substring matching queries.

Latency:

The latency for a prefix B-tree is generally low because it can quickly narrow down the search space by using prefixes.
However, as the search term grows longer, the number of relevant paths increases, which might impact performance slightly.

Freshness:

Freshness is highly dependent on how often new data is added and whether the index is updated promptly. Efficient implementation can maintain good freshness with minimal delays.
The index itself is designed for quick updates and deletions, making it relatively easy to keep fresh.

Ops Cost:

The ops cost is moderate as well. It involves maintaining an efficient B-tree structure that requires periodic rebalancing but does not require the complex setup of a full-text search service.
However, managing the prefix tree can become challenging with very large datasets or frequent updates.
Delegating to a Search Service

Using a dedicated search service like Elasticsearch is another option. These services are highly optimized for text searches and offer advanced features out-of-the-box.

Latency:

The latency provided by a search service is typically excellent because they are designed for high-speed queries.
However, the performance can be impacted if the service itself has high-latency or limited resources.

Freshness:

Freshness is generally very good as long as the search service receives real-time updates. Modern services like Elasticsearch offer features such as indexing pipelines and near-real-time capabilities to ensure data freshness.
The challenge here lies in setting up and maintaining a robust integration with the database to keep the index synchronized.

Ops Cost:

The ops cost is generally high for delegating to a search service due to the additional infrastructure required. This includes setting up, configuring, scaling, and maintaining the search service itself.
However, this might be offset by the operational simplicity provided by managed services like AWS OpenSearch or Google Cloud Search.

#### Closing Summary

For "search as you type," each method has its strengths:

Trigram Index is simple to implement and maintain but may have some latency overhead for fuzzy matching.
Prefix B-Tree offers good performance with moderate ops cost, suitable for precise substring queries.
Delegating to a Search Service provides the best experience in terms of freshness and speed but comes at a higher operational cost.

Choosing the right solution depends on specific requirements such as data size, update frequency, and tolerance for complexity. For smaller datasets or lower ops budgets, trigram indexes or prefix B-trees might be more suitable. For large-scale applications with high freshness requirements, delegating to a search service could be a better choice despite higher costs.

#### Common Mistakes/Follow-Ups

#### Common mistakes
include underestimating the need for frequent updates and not considering the trade-offs between latency and accuracy in trigram indexes. Follow-ups might involve more specific questions about data size, expected load, or desired features.

**Comparison:**

```
Index Type	Latency (ms)	Freshness (s)	Ops Cost
Trigram Index	<50	Real-time	Moderate
Prefix B-Tree	<=100	Real-time	Moderate
Search Service	<50	Real-time	High
```

**Code:**

```

-- Example SQL to create a trigram index CREATE EXTENSION IF NOT EXISTS pg_trgm; CREATE INDEX idx_products_name_trigram ON products USING gist (name gin_trgm_ops);

CODE
Copy

```python
# Example of using Elasticsearch in Python
from elasticsearch import Elasticsearch

es = Elasticsearch([{'host': 'localhost', 'port': 9200}])
response = es.search(index="products", body={"query": {"match": {"name": "search"}}})
print(response)
```

**Example input / output:**

```text
Input:
- Search term: "gol"

Output:
- Suggested matches:
  - Golf Balls
  - Golfing Gear

Explanation:
- Trigram index would match terms containing "gol" as a trigram.
- Prefix B-tree would find matches starting with "gol".
- Elasticsearch would return relevant documents based on the best matching terms.
```

**Data:**

```json
{
  "product_id": 1,
  "name": "Golf Balls",
  "description": "High-quality golf balls for better performance on the course."
}
```

**Flow:**

1. B
2. Prefix B-Tree
3. Elasticsearch
4. Search Results

