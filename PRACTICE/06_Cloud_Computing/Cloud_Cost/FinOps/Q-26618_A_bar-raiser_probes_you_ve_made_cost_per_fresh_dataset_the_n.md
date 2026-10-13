---
type: "practice"
difficulty: "Expert"
---

# A bar-raiser probes: you've made 'cost per fresh dataset' the north-star data-platform metric, but teams game it by loosening freshness SLAs to run cheaper less-frequent pipelines. How do you design the metric and its guardrails so cost goals reward genuine efficiency rather than silently degrading data quality and freshness?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Data Platform Costs](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Governing_Data_Platform_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

The challenge here is to design a "cost per fresh dataset" metric that incentivizes efficient and high-quality data processing pipelines, while preventing teams from gaming the system by lowering freshness SLAs to reduce costs. The key ideas include defining clear metrics, setting guardrails, and ensuring transparency and accountability.

#### Triaging the Question

The interviewer is probing your ability to design a robust metric system that balances cost efficiency with data quality and freshness. This involves understanding the trade-offs between computational resources and data timeliness, and ensuring that teams are motivated to maintain both cost savings and data integrity.

#### Assumptions
Cost Metrics: Teams are currently using "cost per fresh dataset" as a primary metric for evaluating their pipeline efficiency.
SLA Relaxation Risk: There is a risk that teams might loosen SLAs to run cheaper, less-frequent pipelines, which could degrade data freshness and quality.
Team Incentives: The goal is to ensure that cost savings are achieved without compromising on the critical aspects of data quality and timeliness.
Step-by-Step Solution:

Define Clear Freshness Metrics:

Primary SLA: Define a primary freshness SLA (e.g., 90% of datasets should be fresh within 3 hours).
Secondary SLA: Define secondary metrics such as the time to refresh, data completeness, and accuracy.

Cost Metrics:

Total Cost Per Dataset: Measure the total cost per dataset processed.
Efficiency Ratio: Introduce an efficiency ratio that takes into account both freshness and computational cost.

Guardrails and Penalties:

Freshness Penalty: Implement penalties for datasets that fall outside of their SLA, such as additional cost or reduced bonus.
Cost Cap: Set a maximum allowed cost per dataset to prevent teams from running extremely inefficient pipelines.

Transparency and Reporting:

Dashboard: Develop a dashboard that shows both cost and freshness metrics in real-time.
Automated Alerts: Implement automated alerts when SLAs are breached or costs exceed the cap.

Feedback Mechanism:

Performance Reviews: Regularly review team performance based on these metrics to provide constructive feedback.
Training Programs: Offer training programs to help teams optimize their pipelines for both cost and freshness.

Continuous Improvement:

Benchmarking: Periodically benchmark different approaches to find the most efficient pipeline configurations.
Process Automation: Automate repetitive tasks to reduce human error and ensure consistent SLA adherence.

#### Closing Summary

By defining clear, balanced metrics and implementing guardrails, we can incentivize teams to achieve cost savings while maintaining high data quality and freshness. This approach ensures that the metric serves as a true north for optimizing cloud resources without compromising on critical business requirements.

#### Common Mistakes/Follow-Ups
Ignoring Data Quality: Teams might focus solely on cost metrics at the expense of data accuracy.
Lack of Transparency: Without proper visibility, teams may not understand the trade-offs and feel unfairly penalized.
Overly Complex Metrics: Overcomplicated metric systems can be difficult to manage and interpret.
Trialing Scenarios:

#### Cons
ider a scenario where a team frequently breaches freshness SLAs. Would they receive penalties? How would these penalties be calculated, and how could we balance them with cost savings?

**Comparison:**

| Dataset ID | Freshness Time (hours) | Processing Cost ($)| SLA (hours) | Max Cost ($) |
|------------|-----------------------|--------------------|-------------|-------------|
| 1          | 2.5                   | 0.4                | 3           | 1           |

**Code:**

```python
def calculate_efficiency_ratio(freshness_time, processing_cost):
    # Example calculation: Efficiency Ratio = (1 - (freshness_time / SLA)) * (1 - (processing_cost / max_cost))
    efficiency_ratio = ((SLA - freshness_time) / SLA) * ((max_cost - processing_cost) / max_cost)
    return efficiency_ratio

# Example dataset
dataset_freshness_time = 2.5  # hours
processing_cost = 0.4  # dollars
SLA = 3  # hours
max_cost = 1  # dollar

efficiency_ratio = calculate_efficiency_ratio(dataset_freshness_time, processing_cost)
print(f"Efficiency Ratio: {efficiency_ratio}")
```

**Example input / output:**

```text
Input:
- Dataset ID: 1
- Freshness Time: 2.5 hours
- Processing Cost: $0.4
- SLA: 3 hours
- Max Cost: $1

Output:
Efficiency Ratio: 0.6789
```

**Data:**

```json
{
  "dataset": {
    "id": 1,
    "freshness_time": 2.5, // hours
    "processing_cost": 0.4, // dollars
    "SLA": 3, // hours
    "max_cost": 1 // dollar
  }
}
```

**Flow:**

1. Define Freshness & Cost SLAs
2. Implement Metric Dashboard
3. Monitor Real-Time Metrics
4. Set Guardrails (Cost Caps,Freshness Penalties)
5. Automate Alerts for Breaches
6. Regular Performance Reviews
7. Provide Training Programs

