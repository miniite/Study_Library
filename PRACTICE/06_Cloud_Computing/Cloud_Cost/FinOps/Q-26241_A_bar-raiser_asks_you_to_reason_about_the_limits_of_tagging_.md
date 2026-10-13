---
type: "practice"
difficulty: "Expert"
---

# A bar-raiser asks you to reason about the limits of tagging: you've said allocation degrades gracefully below 100% tag coverage. Formalize that — what's the minimum tag coverage below which your allocation number becomes indefensible, and how do you attribute the untaggable remainder without picking arbitrary winners?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shared Cost Attribution](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Shared_Cost_Attribution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a DevOps Architect/Principal context, we're discussing FinOps and cloud cost governance, where tagging is used to allocate costs to specific departments or projects within an organization. The bar-raiser's question probes the limits of this practice, specifically asking when 100% tag coverage becomes unachievable and how to handle the remaining untagged expenses.

#### Triage

The core of this question revolves around understanding the practical limitations of tagging in cost allocation. We need to establish a threshold for minimum tag coverage below which allocations become indefensible. Additionally, we must consider how to attribute the remainder of costs without resorting to arbitrary methods that could introduce biases or misrepresentations.

#### First-Principles Solution

Tag Coverage Threshold:

Tag coverage refers to the percentage of total cloud spend that can be accurately attributed to specific projects or departments.
A 90% tag coverage threshold is often considered sufficient in many organizations, as it allows for a reasonable level of cost visibility and allocation accuracy.

Defensible Allocation:

Below 90% tag coverage, the reliability and defensibility of cost allocations become questionable due to insufficient data points.
When tag coverage falls below 70%, the remaining costs are often considered 'untaggable' or 'orphans'.

Attribution Methods for Untagged Costs:

Historical Patterns: Use historical spending patterns to attribute untagged costs based on previous distributions.
Resource Proportions: Allocate untagged costs proportionally based on the size or usage of resources across departments/projects.
Budget Allocations: Distribute untagged costs according to pre-defined budget allocations, assuming a certain level of spending by each department.

#### Closing Summary
and Common Mistakes/Follow-Ups
The minimum tag coverage threshold below which allocations become indefensible is around 70%. Below this, the remaining 30% (untaggable) must be managed through non-arbitrary methods.
Key mistakes include:
Arbitrarily assigning untaggable costs to a single department or project.
Ignoring historical spending patterns and resource proportions in favor of simpler but less accurate methods.
Common Follow-Ups
How do you balance the need for granular cost visibility with practical limitations?
Can machine learning be used to improve tagging accuracy, especially for untaggable costs?

**Comparison:**

| Tag Coverage Threshold | Description                           |
|------------------------|---------------------------------------|
| >=90%                  | Defensible Allocations                |
| 70%-89%               | Attribution Methods for Untagged Costs|
| <70%                   | Indefensible, Arbitrary Allocation    |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Tag Coverage] --> B{90%+ Tagged};
    B --> C[Defensible Allocation];
    A --> D{70% - 89% Tagged};
    D --> E[Attribution Methods for Untagged Costs];
    E --> F[Historical Patterns, Resource Proportions, Budget Allocations];
```

**Example input / output:**

```text
Input:
Total Spend: $1,000,000
Tagged Spend: $900,000 (90%)
Untagged Spend: $100,000 (10%)

Output:
- Defensible Allocations for Tagged Costs
  - Department A: $450,000 (50%)
  - Department B: $360,000 (40%)
  - Department C: $80,000 (9%)
  - Department D: $120,000 (13%)

- Attribution for Untagged Costs:
  - Historical Patterns: Allocate based on previous year's spending.
  - Proportional Allocation: Use resource sizes across departments.
```

**Data:**

```yaml
tag_coverage_thresholds:
  - name: Defensible Allocation
    threshold: 90%
    description: Sufficient tag coverage for defensible cost allocation.
  - name: Untaggable Costs
    threshold: 70%
    description: Remaining costs that cannot be tagged and require alternative attribution methods.
```

**Flow:**

1. Total Spend
2. Tag Coverage > 90%
3. Defensible Allocations
4. Tag Coverage <= 70%
5. Attribution Methods forUntagged Costs
6. Historical Patterns,Proportional Allocation,Budget Allocations

