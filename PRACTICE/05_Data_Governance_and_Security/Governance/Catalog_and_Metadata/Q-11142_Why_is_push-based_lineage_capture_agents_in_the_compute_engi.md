---
type: "practice"
difficulty: "Deep"
---

# Why is push-based lineage capture (agents in the compute engines) fundamentally different in reliability and coverage from pull/parse-based lineage, and when would you accept the weaker one?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Scaling Lineage Graphs](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Scaling_Lineage_Graphs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When we're dealing with data catalog and metadata engineering, push-based lineage capture (where agents are placed within compute engines to log changes) is fundamentally different from pull/parse-based lineage in terms of reliability and coverage. Push-based lineage offers a more reliable and comprehensive view of the data transformations, whereas pull/parse-based relies on parsing external logs or metadata, which can be less consistent and thorough.

Let's break down why push-based lineage is superior:

Reliability: Agents within compute engines can capture lineage immediately as data flows through them, without any latency. This ensures that every transformation step is logged accurately.
Coverage: Push-based agents can monitor all steps in the pipeline, including those performed by complex or non-traditional tools (like machine learning models), which might not generate external logs.

However, there are scenarios where pull/parse-based lineage might be accepted due to constraints such as cost or technical feasibility:

Cost and Complexity: Implementing agents within every compute engine can be resource-intensive. If the organization has limited resources, parsing existing metadata and logs might be more feasible.
Technical Feasibility: Not all systems may support agent installation, especially legacy or third-party tools.

In summary, while push-based lineage is generally preferred for its reliability and comprehensive coverage, pull/parse-based can be a viable alternative in constrained environments.

**Comparison:**

```
Lineage Capture Method	Reliability	Coverage
Push-Based	High	Complete
Pull/Parse-Based	Low	Incomplete
```

**Code:**

```python
# Example of a simple push-based lineage capture in Python (using a mock agent)
class DataTransformationAgent:
    def __init__(self, data):
        self.data = data

    def transform(self):
        # Simulate transformation logic
        transformed_data = self.data * 2
        self.log_lineage()
        return transformed_data

    def log_lineage(self):
        print(f"Transformed from {self.data} to {transformed_data}")

# Example usage
data_agent = DataTransformationAgent(10)
transformed_data = data_agent.transform()
```

**Example input / output:**

```text
# Example of input-output for push-based lineage capture
initial_data = [1, 2, 3]
transformations = [
    ("data_cleaning", {"min_value": 0}),
    ("aggregation", {})
]

final_result = initial_data

for transformation in transformations:
    agent = DataTransformationAgent(final_result)
    transformed_data = agent.transform()
    final_result = transformed_data
```

**Data:**

```json
{
  "source": "sales_raw",
  "operations": [
    {
      "name": "data_cleaning",
      "input": ["sales_raw"],
      "output": "cleaned_sales"
    },
    {
      "name": "aggregation",
      "input": ["cleaned_sales"],
      "output": "monthly_sales_totals"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Push-Based Lineage
6. Is Agent Installed?
7. Capture Data TransformationImmediately
8. Pull Metadata from ExternalSources
9. Pull/Parse-Based Lineage
10. Are Resources Limited?
11. Accept Weak Coverage forCost
12. Implement ReliablePush-Based Lineage

