---
type: "practice"
difficulty: "Deep"
---

# A dashboard needs data joined across two domain-owned products in a mesh. Do you replicate their product into your area or federate-query it live? How do you decide?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cross-domain Joins in a Mesh](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Cross-domain_Joins_in_a_Mesh.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether to replicate or federate-query data from another domain's product, we need to consider several factors including performance, cost, and data freshness. Replicating data into our own area ensures faster queries but can introduce latency and duplication overhead. On the other hand, federated querying maintains real-time data integrity at a potential cost of slower query times.

The core decision hinges on understanding the nature and frequency of data updates, as well as the specific requirements for performance and consistency in the dashboard. We will reason through this from first principles to arrive at an informed decision.

**Comparison:**

| Scenario       | Decision         | Reasoning                                                       |
|----------------|------------------|----------------------------------------------------------------|
| High-frequency updates, low-latency requirements | Replicate        | Minimizes query latency at the cost of increased data redundancy. |
| Low-frequency updates, high-latency tolerance   | Federate-Query   | Ensures real-time accuracy without the overhead of replication.  |

**Code:**

```python
# Example Python pseudo-code to illustrate the logic
def decide_replication_or_federation(product_a, product_b):
    if product_a.update_frequency > product_b.update_frequency:
        return "federate-query"
    elif performance_requirements["response_time"] < 100ms:
        return "replicate"
    else:
        return "federate-query"
```

**Example input / output:**

| Input | Decision       |
|-------|----------------|
| Product A update frequency = hourly, rows = 50K, response time requirement < 100ms | Replicate |
| Product B update frequency = daily, rows = 30K, response time requirement > 120ms | Federate-Query |

**Data:**

```json
{
  "productA": {
    "updateFrequency": "hourly",
    "numRows": 50000,
    "performanceRequirements": {
      "responseTime": 120ms
    }
  },
  "productB": {
    "updateFrequency": "daily",
    "numRows": 30000,
    "performanceRequirements": {
      "responseTime": 50ms
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Analyze Data Frequency andRequirements
4. Is update frequency high?
5. Replicate data for fasterqueries
6. Federate-query to maintainreal-time accuracy
7. Monitor performance, adjust ifnecessary
8. Ensure consistent and timelyupdates

