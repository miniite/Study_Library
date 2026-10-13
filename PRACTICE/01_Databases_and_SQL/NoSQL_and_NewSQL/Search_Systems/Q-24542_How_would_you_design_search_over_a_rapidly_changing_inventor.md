---
type: "practice"
difficulty: "Deep"
---

# How would you design search over a rapidly changing inventory where stock counts update every second? Discuss update volume, refresh cost, and separating volatile fields from stable ones.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Search Index Internals and Mappings](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an efficient search system for a rapidly changing inventory that updates stock counts every second, we need to balance between high-frequency updates, minimizing refresh costs, and ensuring that only relevant data is indexed. We'll approach this problem by separating volatile fields (like real-time stock counts) from stable ones (such as product descriptions or categories).

TL;DR: The key is to use a hybrid indexing strategy: maintain an Elasticsearch cluster for volatile fields and another for more stable data, with periodic re-indexing of the latter. This minimizes overhead while ensuring timely search results.

#### Triage

The question asks us to design a search system for inventory updates that happen frequently (every second). The core aspects to consider are:

Update Volume: How many records need updating per second.
Refresh Cost: The cost in terms of processing power and memory when Elasticsearch performs refresh operations.
Volatile vs. Stable Fields: Determining which fields change rapidly and which remain relatively constant.
First Principles

Understanding Elasticsearch Refresh Mechanism:

Elasticsearch performs a refresh operation, which triggers indexing updates, to ensure that recent changes are available for search queries.
By default, the refresh interval is set every 1s (1000ms).

Impact of High-Volume Updates on Refreshes:

Frequent stock count updates can lead to many small index operations, increasing the load and potentially causing performance issues.

Separation of Concerns:

Separate volatile fields from stable ones to optimize indexing frequency.
Volatile fields should be indexed more frequently but at a lower cost per update.
Stable fields should be indexed less often but with higher data integrity.

#### Solution

Elasticsearch Cluster Design:

Index A: For volatile fields (e.g., stock count, availability status).
Set refresh interval to 1s or even shorter if necessary.
Use a separate index for frequent updates.
Ensure this index has adequate resources allocated.
Index B: For stable fields (e.g., product name, category, description).
Set refresh interval to a longer period (e.g., every 30 minutes or hourly).
This reduces the number of indexing operations and associated costs.

Hybrid Indexing Strategy:

Use a multi-index strategy where volatile fields are indexed in real-time but with lower priority, while stable data is refreshed less frequently.
Leverage Elasticsearch’s index API to dynamically manage which fields are updated more often.

Monitoring and Optimization:

Implement monitoring tools (e.g., Kibana) to track refresh performance and resource usage.
Adjust refresh intervals based on observed load and performance metrics.
Use machine learning models to predict spikes in update volume and proactively manage resources.

Batching Updates:

Implement batch processing for updating stable fields to reduce the number of indexing operations during periods of high activity.
Use background processes or scheduled jobs to update these indexes at intervals that minimize performance impact.

#### Closing Summary

By separating volatile and stable fields, we can optimize our Elasticsearch setup to handle rapid updates efficiently. This approach minimizes refresh costs while ensuring timely search results for frequently changing data. Regular monitoring and optimization will help maintain the balance between responsiveness and resource utilization.

#### Common Mistakes/Follow-Ups
Ignoring Refresh Costs: Many developers overlook the overhead of frequent indexing operations, leading to degraded performance.
Over-Batching Updates: While batching can reduce costs, overly aggressive batch processing might delay important updates.
Failing to Monitor Performance: Without continuous monitoring, it's easy to miss signs of resource contention or inefficiencies.
Triaging Specifics

The key assumptions are:

The stock count changes frequently and critically affect search results.
Product descriptions and other metadata change less often but still need up-to-date indexing.
Resources (CPU, memory) are limited and must be managed carefully.

**Comparison:**

```
Index Type	Field Type	Refresh Interval
Volatile	Stock Count	1 second
Stabile	Metadata	30 minutes
```

**Code:**

```python
# Example code snippet for dynamic refresh settings in Elasticsearch
from elasticsearch import Elasticsearch

es = Elasticsearch()

def update_volatile_fields(stock_counts):
    for product_id, count in stock_counts.items():
        es.update(index="volatile-field-index", id=product_id, body={"doc": {"stock_count": count}})

def update_stable_fields(products):
    for product_id, details in products.items():
        es.update(index="stable-field-index", id=product_id, body={"doc": {"name": details["name"], "category": details["category"]}})

# Example usage
volatile_updates = {1: 10, 2: 5}
update_volatile_fields(volatile_updates)

stable_updates = {1: {"name": "Widget", "category": "Gadgets"}, 2: {"name": "Gadget", "category": "Widgets"}}
update_stable_fields(stable_updates)
```

**Example input / output:**

```text
Input: Stock count updates for products [1, 2]
Output:
- volatile-field-index updated with new stock counts.
- stable-field-index remains unchanged due to less frequent updates.

Input: Metadata update for product 1 (name: Widget -> NewWidget)
Output:
- stable-field-index updated with new metadata details.
- volatile-field-index remains unchanged as it only handles stock count changes.
```

**Data:**

```json
{
    "volatile-field-index": [
        {"_id": 1, "stock_count": 10},
        {"_id": 2, "stock_count": 5}
    ],
    "stable-field-index": [
        {"_id": 1, "name": "Widget", "category": "Gadgets"},
        {"_id": 2, "name": "Gadget", "category": "Widgets"}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Receive Product Updates
4. Are the updates volatile?
5. Volatile Field Index Update
6. Stable Field Index Update
7. Maintain Real-Time Data
8. Batch Process Updates
9. Monitor Performance

