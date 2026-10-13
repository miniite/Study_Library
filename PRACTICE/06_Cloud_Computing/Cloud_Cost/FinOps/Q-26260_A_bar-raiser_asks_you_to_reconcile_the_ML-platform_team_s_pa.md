---
type: "practice"
difficulty: "Expert"
---

# A bar-raiser asks you to reconcile the ML-platform team's paved-road (managed training, managed serving, managed feature store) with the fact that the managed premium now shows up as a top cost line finance is questioning. Build the honest TCO of the managed ML platform versus teams self-managing, including platform-team headcount.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ TCO of Self-managed Platforms](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/TCO_of_Self-managed_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In this scenario, we need to reconcile why the managed premium costs attributed to the ML-platform team are showing up as a significant cost line in finance, despite their use of a "paved-road" approach (managed training, serving, and feature store). We will calculate the Total Cost of Ownership (TCO) for both self-managing teams and using the managed platform. Additionally, we'll include an analysis of headcount implications. This requires a detailed financial breakdown, including direct costs, indirect costs, and the cost of managing the platform.

#### Triaging the Question

The interviewer is probing our ability to:

Understand the concept of TCO in cloud environments.
Analyze managed vs self-managed services for ML platforms.

#### Cons
ider headcount impacts and their financial implications.
Present a comprehensive, nuanced solution that covers both technical and financial aspects.
Step-by-Step Solution
1. Define Key Metrics

We need to define the key metrics:

TCO Components: Direct costs (e.g., compute, storage), indirect costs (e.g., management overhead, support).
Managed Platform Costs: Compute, storage, data engineering, and platform management.
Self-Managing Teams Costs: Compute, storage, infrastructure management, data engineering.
2. Estimate Managed Platform Costs

Let's assume the managed platform is AWS SageMaker:

Compute (EC2/Spot Instances): $0.14 per hour for ML training.
Storage (S3): $0.023 per GB/month.
Feature Store: Assume a monthly fee of $500.
3. Estimate Self-Managing Team Costs

For self-managing teams, consider the following:

Compute and Storage on AWS EC2/EC2:
Training: $0.14 per hour.
Inference: $0.006 per request.
Storage (S3): $0.023 per GB/month.
Data Engineering: Assume a data engineer’s salary of $75,000/year.
4. Platform Management Team Costs

Assume the managed platform team has 10 members:

Salaries and Benefits: Average salary of $120,000/year, benefits (health insurance, etc.) at 30%.
Total Annual Cost for Platform Team: (10 \times ($120,000 + $120,000 \times 0.3) = $1,560,000).
5. Detailed TCO Calculation

For a hypothetical project:

Project Duration: 1 year.
Number of Training Hours: 5,000 hours.
Number of Inference Requests: 2 million.

Managed Platform Costs:

Compute (Training): (5,000 \times 8 \text{ hours/day} \times $0.14 = $56,000).
Storage: Assuming 1 GB per training job, (5,000 \times $0.023 = $1,150).
Feature Store: $500/month (\times 12 = $6,000).

Total for managed platform: ( $56,000 + $1,150 + $6,000 + $1,560,000 = $1,633,200 ).

Self-Managing Team Costs:

Compute (Training): $56,000.
Storage: (2 million requests \times 0.1 GB/request \times $0.023 = $55,700).
Data Engineering: Assuming 10% of the data engineer’s time is dedicated to this project, cost is $75,000/year.

Total for self-managing team: ( $56,000 + $55,700 + $75,000 = $186,700 ).

6. Summary and Headcount Implications
Total Managed Platform Cost: $1,633,200.
Self-Managing Team Cost: $186,700.

The managed platform is significantly more expensive but provides a fully managed service with less overhead for the teams. The platform team’s headcount investment must be justified by factors such as project complexity, scalability, and support needs.

#### Closing Summary

In conclusion, while the managed ML platform incurs higher TCO due to direct costs and the presence of a dedicated management team, it offers a more scalable and supported environment. The decision should consider the balance between cost savings from self-managing versus the benefits of having a fully managed service with minimal management overhead.

#### Common Mistakes/Follow-Ups
Overlooking Indirect Costs: Ensuring to include indirect costs such as management, support, and data engineering in the TCO.
Not Accounting for Scalability: The managed platform can scale more easily without additional investment from teams.
Ignoring Support and Maintenance: Managed services often come with built-in support and maintenance, which self-managing teams may need to provide themselves.

If asked further, we could discuss:

Automated Cost Optimization: Implementing tools like AWS Cost Explorer or similar for the managed platform.
Cloud Provider Rebates: Considering any potential rebates from cloud providers that might offset some costs.
Cost vs. Value Proposition: Evaluating whether the higher cost is justified by reduced operational complexity and increased reliability.

**Comparison:**

```
Component	Managed Platform Cost ($)	Self-Managing Team Cost ($)
Compute (Training)	56,000	56,000
Storage	1,150	55,700
Feature Store	6,000	-
Platform Team Salary	-	-
Total TCO	1,633,200	186,700
```

**Code:**

```python
# Example Python code for calculating TCO

def calculate_tco_managed(platform_cost):
    return platform_cost + 1560000  # Platform team salary cost

managed_platform_cost = (calculate_tco_managed(56000) +
                         1150 +
                         6000)
print(f"Managed Platform TCO: ${managed_platform_cost}")
```

**Example input / output:**

```text
Input:
- Training Hours: 5,000
- Inference Requests: 2 million
- Storage Per Request (GB): 0.1

Output:
- Managed Platform Compute Cost: $56,000
- Managed Platform Storage Cost: $1,150
- Managed Feature Store Cost: $6,000
- Total Platform Team Annual Salary: $1,560,000
- Managed Platform TCO: $1,633,200

Self-Managing Team Costs:
- Compute Training Cost: $56,000
- Storage Cost: $55,700
- Data Engineering Cost: $75,000
- Total Self-Managing Team TCO: $186,700
```

**Data:**

```json
{
    "projectDetails": {
        "training_hours": 5000,
        "inference_requests": 2000000,
        "storage_per_request_gb": 0.1,
        "compute_cost_per_hour_training": 0.14,
        "storage_cost_per_gb_monthly": 0.023,
        "feature_store_monthly_fee": 500,
        "platform_team_salary_annual": 120000,
        "benefits_percentage": 0.3
    }
}
```

**Flow:**

1. Yes
2. No
3. Define Project Details
4. Is Managed Service CostHigher?
5. Calculate TCO forSelf-Managing Team
6. Consider Scalability andSupport Benefits
7. Compare Costs: Managed vs.Self-Managed
8. Justify Platform TeamHeadcount

