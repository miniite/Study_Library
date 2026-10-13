---
type: "practice"
difficulty: "Deep"
---

# A bar-raiser probes: you've built a data catalog and lineage system, and it's now scanning every table's metadata continuously plus running expensive profiling. Justify the cost of the governance/lineage tier against its value, and defend which lineage granularity is non-negotiable versus a tunable expense.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Data Platform Costs](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Governing_Data_Platform_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To justify the cost of our data catalog and lineage system in the context of FinOps and cloud cost governance, we need to assess its value against the costs involved. Our goal is not just to document metadata but also to derive actionable insights that drive business efficiency and compliance.

Firstly, let's summarize the current state: We are scanning every table’s metadata continuously and running expensive profiling. The primary benefits of this system include improved data governance, enhanced lineage, and deeper understanding of our data assets. However, there is a trade-off between these benefits and operational costs.

#### Triage

The core question here is whether the cost of maintaining a fully-fledged lineage system with continuous scanning and expensive profiling justifies its value. The interviewer is probing for an in-depth analysis that considers both the business needs and technical constraints.

#### Assumptions
Cost: We assume a cloud environment where ongoing scans and profiles incur significant costs.
Benefits: We recognize benefits like improved data governance, enhanced compliance, and better data management practices.
Scalability: The system must be scalable to handle growing datasets and evolving business needs.

#### First-Principles Solution
Non-Negotiable Lineage Granularity

The non-negotiable lineage granularity is the transaction level for financial and regulatory reasons. Every financial transaction should have a clear lineage from raw data ingestion to final reporting, ensuring compliance with regulations such as GDPR or SOX. This is crucial because:

Regulatory Compliance: Transactions must be traceable.
Audit Trail: Detailed records are required for audits.
Tunable Expenses

For other data types and use cases, we can consider a tunable approach where the granularity of lineage can be adjusted based on cost-benefit analysis. For instance:

Historical Data: Lower granularity (e.g., daily or weekly summaries) might suffice.
Non-Critical Business Processes: Less frequent scanning (e.g., once a day instead of every hour).
Justification Steps
Impact Assessment: Evaluate the impact on key business processes and compliance requirements. For critical transactions, the cost is justified by regulatory compliance and audit needs.
Cost-Benefit Analysis: Perform a detailed cost-benefit analysis for different data types. For example, financial transaction lineage might have higher costs but also generate significant value through improved governance and reduced risk.
Performance Optimization: Implement strategies to optimize performance without compromising critical lineage requirements. Techniques like caching, smart sampling, and incremental updates can help reduce costs.

#### Closing Summary

In conclusion, the cost of our data catalog and lineage system is justified by its critical role in ensuring regulatory compliance and improving data governance. However, we must adopt a tunable approach for less critical datasets to balance cost with business needs. The transaction-level granularity is non-negotiable due to regulatory requirements, while other granularities can be adjusted based on a balanced cost-benefit analysis.

#### Common Mistakes/Follow-Ups
Over-Engineering: Failing to identify and prioritize critical use cases can lead to unnecessary costs.
Ignoring Compliance Risks: Treating all data with the same granularity without considering regulatory requirements can result in significant legal issues.
Insufficient Performance Optimization: Not implementing performance optimizations can lead to suboptimal resource utilization.

**Comparison:**

```
Data Type	Granularity	Scan Frequency
Financial	Transaction	Every minute
Historical	Daily summary	Once a day
Non-Critical	Summary	Once a week
```

**Code:**

```python
# Example of a simple cost-benefit analysis function
def cost_benefit_analysis(data_type, scan_frequency):
    costs = {"high": 5000, "low": 1000}
    benefits = {"high": 20000, "low": 4000}

    if data_type == "transaction":
        return (costs["high"], benefits["high"])
    else:
        return (costs[scan_frequency], benefits[scan_frequency])

# Example usage
print(cost_benefit_analysis("transaction", "high"))
```

**Example input / output:**

```text
Input:
- Data type: financial, Scan frequency: every_minute

Output:
- Costs: $5000
- Benefits: $20000

Justification: The high granularity of transaction-level lineage is non-negotiable for regulatory compliance and audit needs. The cost-benefit analysis shows a significant positive net value.
```

**Data:**

```json
{
  "data_types": {
    "financial": {"granularity": "transaction"},
    "historical": {"granularity": "daily_summary"}
  },
  "scans": [
    {"type": "transaction", "frequency": "every_minute"},
    {"type": "non_critical", "frequency": "once_a_day"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Transaction Lineage
4. Is data critical?
5. High Granularity
6. Tunable Granularity
7. Regulatory Compliance &Audit Trail
8. Cost-Benefit Analysis

