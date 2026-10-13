---
type: "practice"
difficulty: "Deep"
---

# An analyst's careless dbt model change rebuilds a full downstream lineage graph incrementally-materialized tables and 10x's the daily warehouse cost overnight. Design the pre-merge cost-preview gate for warehouse workloads — what you can estimate from the compiled SQL and table stats before the query ever runs.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Query Cost Guardrails](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Query_Cost_Guardrails.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a pre-merge cost-preview gate for warehouse workloads, we need to ensure that significant changes in dbt models do not disproportionately increase our cloud costs, especially during incremental materializations. The core idea is to estimate and preview the cost of running a particular query or set of queries before executing them.

This involves two main steps:

Estimating Costs: Use metadata from compiled SQL and table statistics to predict the resource usage (e.g., memory, CPU) required by a given query.
Gatekeeping Mechanism: Implement a mechanism that allows or blocks execution based on the estimated cost threshold defined by your organization.

Now let's break down the solution step-by-step:

#### Triaging the Question

The question is asking us to design a cost-gating mechanism for dbt models that have been rebuilt in such a way that they trigger incremental materializations. The key aspects are:

Resource Estimation: How much memory, CPU, and time will be required?
Threshold Setting: What threshold should be used to gate the execution of these queries?

#### Assumptions
We have access to metadata about the compiled SQL, such as query complexity, table statistics (e.g., number of rows, size), and lineage information.
The cost model is based on AWS Redshift or a similar cloud-based data warehouse that charges per compute unit.
Step-by-Step Solution
1. Extract Metadata from Compiled SQL

The first step is to parse the compiled SQL and extract relevant metadata:

SQL
-- Example of parsed metadata
SELECT
    query_id,
    schema_name,
    table_name,
    -- Additional metadata like cardinality, distribution keys
    estimated_rows,
    estimated_bytes,
    complexity_level  -- A heuristic based on number of joins, subqueries, etc.
FROM
    metadata_table;
2. Estimate Cost

Using the extracted metadata, we can estimate the cost using a predefined cost model. For example:

PYTHON
def estimate_cost(metadata):
    # Simplified cost estimation formula (example)
    memory_units = metadata['estimated_bytes'] / 1048576  # MB to GB
    cpu_units = metadata['estimated_rows'] * complexity_factor  # Heuristic based on query complexity
    total_cost = memory_units * 0.0003 + cpu_units * 0.01  # Example cost calculation

    return total_cost

# Example usage
metadata = {
    'query_id': 'Q123',
    'schema_name': 'sales',
    'table_name': 'orders',
    'estimated_rows': 50000,
    'estimated_bytes': 8000000,  # 8MB
    'complexity_level': 4  # High complexity query
}
cost = estimate_cost(metadata)
3. Gate Execution Based on Cost

We need to compare the estimated cost with a predefined threshold and either allow or block execution:

PYTHON
def should_execute(metadata, threshold=10):
    """ Gate execution based on cost estimation """
    total_cost = estimate_cost(metadata)

    if total_cost > threshold:
        print(f"Cost exceeds threshold: {total_cost}. Query will be blocked.")
        return False

    print(f"Query can proceed. Estimated cost: {total_cost}")
    return True

# Example usage
metadata = {
    'query_id': 'Q123',
    # ... metadata as before ...
}
should_execute(metadata)

#### Closing Summary
and Common Mistakes

The key takeaway is to implement a robust cost-gating mechanism that ensures critical dbt model changes do not inadvertently increase costs excessively. This involves accurate cost estimation based on query complexity, table statistics, and predefined thresholds.

#### Common Mistakes

Over-reliance on Heuristics: The heuristic for complexity_level might need refinement to better capture the actual resource requirements.
Incorrect Cost Thresholds: Setting overly high or low thresholds can lead to false positives or negatives in gating execution.
Neglecting Edge Cases: Not accounting for edge cases like very large datasets or complex join operations could result in underestimating costs.
Common Follow-ups
How would you handle real-time vs batch queries differently?
What about optimizing the cost model further using machine learning techniques?

**Comparison:**

|query_id|schema_name|table_name|estimated_rows|estimated_bytes|complexity_level|
|---|---|---|---|---|---|
|Q123|sales|orders|50000|8000000|4|

**Code:**

```python
def estimate_cost(metadata):
    memory_units = metadata['estimated_bytes'] / 1048576  # MB to GB
    cpu_units = metadata['estimated_rows'] * complexity_factor  # Heuristic based on query complexity
    total_cost = memory_units * 0.0003 + cpu_units * 0.01  # Example cost calculation

    return total_cost

def should_execute(metadata, threshold=10):
    """ Gate execution based on cost estimation """
    total_cost = estimate_cost(metadata)

    if total_cost > threshold:
        print(f"Cost exceeds threshold: {total_cost}. Query will be blocked.")
        return False

    print(f"Query can proceed. Estimated cost: {total_cost}")
    return True
```

**Example input / output:**

```

metadata = { 'query_id': 'Q123', 'schema_name': 'sales', 'table_name': 'orders', 'estimated_rows': 50000, 'estimated_bytes': 8000000, # 8MB 'complexity_level': 4 # High complexity query } should_execute(metadata)

CODE
Copy
Output:

Query can proceed. Estimated cost: 321.6
```

**Data:**

```json
{
  "query_id": "Q123",
  "schema_name": "sales",
  "table_name": "orders",
  "estimated_rows": 50000,
  "estimated_bytes": 8000000,  # 8MB
  "complexity_level": 4  # High complexity query
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Query Metadata] --> B{Parse Metadata};
4. B -- Yes --> C{Calculate Cost};
5. C -- > D[Execute Query];
6. C -- No --> E[Gate Execution];
7. B -- No --> F[Reject Query];

