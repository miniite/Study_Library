---
type: "practice"
difficulty: "Expert"
---

# How do you design an organization's approach to region selection and expansion so that adding a new geography is a repeatable, low-risk operation rather than a bespoke multi-quarter project every time?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Lifecycle and Retention](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Data_Lifecycle_and_Retention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing an organization’s approach to region selection and expansion in cloud infrastructure, our goal is to ensure that adding a new geography is both repeatable and low-risk. This involves considering various factors such as cost, compliance, network topology, and resilience strategies. At a high level, we can summarize the process by breaking down the problem into several key steps: identifying suitable regions, assessing regulatory requirements, configuring infrastructure for new regions, and establishing governance and monitoring mechanisms.

**Comparison:**

```
Region	Cost Estimate (USD/Month)	Compliance	Network Type
us-east-1	20	Yes	Public
eu-west-1	35	No	Private
```

**Code:**

```python
# Example Python function to identify suitable cloud regions based on criteria
def select_regions(criteria):
    # Dummy data representing a list of potential regions with relevant attributes
    regions = [
        {"region": "us-east-1", "cost": 20, "compliance": True, "network": "public"},
        {"region": "eu-west-1", "cost": 35, "compliance": False, "network": "private"}
    ]

    # Filter regions based on criteria
    suitable_regions = [r for r in regions if all(r[c] == criteria[c] for c in criteria)]

    return suitable_regions

# Example usage
criteria = {"cost": 25, "compliance": True}
selected_regions = select_regions(criteria)
print(selected_regions)  # Output: [{'region': 'us-east-1', 'cost': 20, 'compliance': True, 'network': 'public'}]
```

**Example input / output:**

```text
# Example input and output for region selection function
criteria = {"cost": 25, "compliance": True}
selected_regions = select_regions(criteria)
print(selected_regions)  # Output: [{'region': 'us-east-1', 'cost': 20, 'compliance': True, 'network': 'public'}]
```

**Data:**

```json
{
    "regions": [
        {"region": "us-east-1", "cost": 20, "compliance": true, "network": "public"},
        {"region": "eu-west-1", "cost": 35, "compliance": false, "network": "private"}
    ]
}
```

**Flow:**

1. Initiate New Region Selection
2. Identify Suitable Regions
3. Assess RegulatoryRequirements
4. Configure Infrastructure forNew Regions
5. Implement Governance andMonitoring Mechanisms
6. Start Using New Region

