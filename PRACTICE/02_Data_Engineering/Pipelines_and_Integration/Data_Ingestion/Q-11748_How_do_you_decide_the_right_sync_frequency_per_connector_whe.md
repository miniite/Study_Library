---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide the right sync frequency per connector when you have 60 connectors and a fixed cost budget?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Incremental Sync and Connector Migration](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Incremental_Sync_and_Connector_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding the right sync frequency per connector with 60 connectors and a fixed cost budget, the key is to balance resource allocation, data freshness, and business needs. This involves understanding the characteristics of each connector (e.g., volume of data, criticality) and aligning that with your available budget.

#### Triaging the Question

The question asks us to determine an optimal sync frequency for 60 connectors within a fixed cost budget. The core is about optimizing resource allocation based on business impact and operational constraints. We need to consider:

Cost per connector: How much does each connector's sync process cost?
Data volume and criticality: Which connectors handle more data, or are they mission-critical?
Business needs: What level of data freshness is required for different connectors?

#### First-Principles Solution

Categorize Connectors:

High-Cost/High-Impact Connectors: These could include critical financial systems, customer relationship management (CRM) tools, or real-time analytics.
Low-Cost/Low-Impact Connectors: Less critical systems with lower data volumes.

Cost Analysis:

Estimate the cost per sync for each connector based on factors like API calls, storage space used, and processing time.
Total costs should not exceed the fixed budget.

Frequency Allocation:

For high-cost/impact connectors, use more frequent (e.g., hourly) syncs to ensure data freshness.
For low-cost/impact connectors, less frequent syncs can be used (e.g., daily).

Optimization Techniques:

Use incremental or delta-based syncing where applicable to reduce overhead.
Implement error handling and retries for critical systems.
Step-by-Step Solution

Gather Data on Connectors:

Collect information on each connector, including the type of data source, expected volume, and business impact.

Estimate Costs:

Calculate the cost per sync for each connector using a tool like Airbyte or Fivetran's API cost estimators.

Prioritize Connectors:

Rank connectors based on their criticality and data volume.
Assign higher priority to more critical systems with larger datasets.

Budget Allocation:

Allocate budget to the top-priority connectors first, ensuring they get sufficient sync frequency.
Rebalance as necessary if costs exceed the fixed budget.

Monitor Performance:

Continuously monitor system performance and adjust frequencies based on business needs and cost constraints.

#### Closing Summary

By categorizing connectors based on their criticality and estimated costs, we can make informed decisions about syncing frequencies. This ensures that high-impact systems receive frequent updates while optimizing the use of a fixed budget for other connectors. Common mistakes include overallocating resources to less important systems or not considering the incremental costs accurately.

#### Common Mistakes/Follow-Ups
Ignoring Criticality: Failing to prioritize critical systems can lead to data inconsistencies.
Over-Engineering Costs: Allocating too much budget to a few connectors might neglect other essential systems.
Lack of Monitoring: Not continuously monitoring and adjusting sync frequencies based on real-time performance metrics.

**Comparison:**

```
Connector	Data Volume (MB)	API Calls	Criticality	Estimated Cost
CRM	100	20	High	$2.8
Email Campaigns	50	30	Medium	$2.95
Inventory	75	15	Low	$2.47
```

**Code:**

```python
# Example cost estimation function
def estimate_sync_cost(connector_info):
    # Placeholder for actual cost calculation logic
    return connector_info['data_volume'] * 0.01 + connector_info['api_calls'] * 0.05

# Sample data schema
connector_data = [
    {"name": "CRM", "data_volume": 100, "api_calls": 20, "criticality": "high"},
    {"name": "Email Campaigns", "data_volume": 50, "api_calls": 30, "criticality": "medium"},
    {"name": "Inventory", "data_volume": 75, "api_calls": 15, "criticality": "low"}
]

# Cost estimation for all connectors
cost_per_connector = {conn['name']: estimate_sync_cost(conn) for conn in connector_data}
```

**Example input / output:**

```text
Input: Cost per connector
Output:
CRM - $2.8
Email Campaigns - $2.95
Inventory - $2.47
```

**Data:**

```yaml
connector-data:
- name: CRM
  data_volume: 100
  api_calls: 20
  criticality: high
- name: Email Campaigns
  data_volume: 50
  api_calls: 30
  criticality: medium
- name: Inventory
  data_volume: 75
  api_calls: 15
  criticality: low
```

**Flow:**

1. Data Collection
2. Categorization
3. Cost Estimation
4. Prioritization
5. Budget Allocation
6. Monitoring & Adjustment

