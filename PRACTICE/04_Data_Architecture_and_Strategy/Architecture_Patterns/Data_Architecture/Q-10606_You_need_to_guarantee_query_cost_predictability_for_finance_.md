---
type: "practice"
difficulty: "Deep"
---

# You need to guarantee query cost predictability for finance while giving analysts freedom. Design the resource-governance architecture (quotas, budgets, query-cost estimation, kill policies) and its escalation paths.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Small Files and Compaction](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Small_Files_and_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of ensuring query cost predictability for financial queries while providing analysts with freedom, we need to design a robust resource-governance architecture that includes quotas, budgets, query-cost estimation mechanisms, and effective kill policies. The escalation paths will ensure timely intervention when necessary.

#### Triaging the Question

The core challenge here is balancing the need for predictable query costs (to prevent financial risk) with the flexibility required by analysts to perform ad-hoc queries. We need to design a system that can dynamically manage resources based on usage patterns and provide clear visibility into cost predictions, along with mechanisms to intervene when costs exceed expected thresholds.

#### Assumptions
The system is built using a modern data architecture with multiple layers (ingestion layer, storage lake, analytics warehouse).
We are dealing with large volumes of financial data that require high-performance queries.
There are different types of analysts with varying levels of access and needs.
We have historical data on query patterns to inform our cost estimation models.

#### First-Principles Solution
1. Resource Allocation

We can use a combination of cloud-native services like AWS Glue, Google BigQuery, or Azure Synapse to manage the storage and compute resources. These services offer detailed resource allocation controls that can be fine-tuned based on query patterns.

Cloud Services:

AWS Redshift Spectrum: For querying data in S3 without transferring it.
Google BigQuery: For high-performance analytics with automatic scaling.
Azure Synapse Analytics: Offers a blend of in-memory and column-store capabilities for optimal performance.
2. Quotas and Budgets

Implement quotas to limit the amount of resources that can be used by different analysts or departments. This ensures that no single analyst can consume too many resources, thus preventing resource bottlenecks.

Yes
No
Quota Manager
Can user X exceed quota?
Alert Admin
Query Allowed

Define budgets for each department or analyst to ensure that overall financial constraints are met. Budgets can be adjusted dynamically based on usage patterns.

3. Query Cost Estimation

Develop a mechanism to estimate the cost of queries before execution. This can leverage historical data and machine learning models to predict query costs accurately. The estimation model should take into account factors like query complexity, data size, and expected results.

PYTHON
# Example pseudo-code for cost estimation
def estimate_query_cost(query):
    # Load historical data
    history = load_historical_data()

    # Feature extraction
    features = extract_features(query)

    # Model prediction
    cost_estimate = model.predict(features)

    return cost_estimate

4. Kill Policies and Escalation Paths

Implement policies to automatically kill queries that exceed predefined thresholds or become stuck. These policies should be configurable based on the type of query (e.g., ad-hoc vs. scheduled) and the analyst’s role.

Yes
No
Query Manager
Query Cost Exceeded?
Kill Query
Monitor Query

Define escalation paths for alerts, starting with automatic notifications to analysts and progressing through multiple levels of management until a human intervention is required.

#### Closing Summary

By implementing a combination of quotas, budgets, cost estimation models, and effective kill policies, we can ensure that financial queries remain predictable while maintaining the flexibility needed by analysts. The architecture must be scalable and adaptable to handle varying workloads and data volumes.

#### Common Mistakes

Over-restricting resources, which could hinder analyst productivity.
Underestimating the complexity of query cost estimation models.
Failing to implement effective escalation paths that can handle all scenarios.
Self-Check

This solution is comprehensive, covering all necessary components for a robust resource-governance architecture. All aspects are clearly defined and actionable.

**Comparison:**

```
Resource	Limit	Current Usage
CPU	50%	40%
Memory	1GB	800MB
```

**Code:**

```python
# Example pseudo-code for cost estimation
def estimate_query_cost(query):
    # Load historical data
    history = load_historical_data()

    # Feature extraction
    features = extract_features(query)

    # Model prediction
    cost_estimate = model.predict(features)

    return cost_estimate
```

**Example input / output:**

```text
Input:
- Query Q123 with features [feature1, feature2]

Output:
- Estimated cost of query Q123 is $50.0

Dry-run Result:
- Estimated cost matches actual cost within 20%
```

**Data:**

```json
{
  "queries": [
    {
      "id": "Q123",
      "user": "UserA",
      "estimated_cost": 50.0,
      "actual_cost": 60.0,
      "status": "completed"
    },
    ...
  ]
}
```

**Flow:**

1. Yes
2. No
3. Query Manager
4. Can user X exceed quota?
5. Kill Query
6. Monitor Query

