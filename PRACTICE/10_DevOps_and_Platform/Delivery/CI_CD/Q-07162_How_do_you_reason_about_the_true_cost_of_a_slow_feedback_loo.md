---
type: "practice"
difficulty: "Expert"
---

# How do you reason about the true cost of a slow feedback loop in economic terms, and what model would you present to leadership to trade CI compute spend against engineering opportunity cost across the whole org?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ CI Performance Investment](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_Cost_Optimization/CI_Performance_Investment.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To reason about the true cost of a slow feedback loop in economic terms, we need to consider both direct and indirect costs associated with reduced productivity and increased risk. In this context, a "slow feedback loop" refers to the time it takes for changes in code or configuration to be observed and validated in production environments. To present this to leadership effectively, we'll model the impact of CI/CD compute spend against engineering opportunity cost across the organization.

#### Triaging the Question

The question is essentially asking how a slow CI/CD feedback loop impacts the overall efficiency and effectiveness of an engineering team. The interviewer wants you to quantify these impacts and propose a trade-off model that balances investment in faster feedback loops with other strategic priorities. Key assumptions include:

The current feedback loop speed.
The cost of compute resources for CI/CD pipelines.
The opportunity cost of wasted engineer time due to slow feedback.

#### First-Principles Solution
Define Metrics: Establish key metrics such as mean time to detect (MTTD) and mean time to repair (MTTR). MTTD measures how long it takes to identify a problem, while MTTR measures the duration until the issue is resolved.
Estimate Economic Impact:
Direct Costs: Compute resources for CI/CD pipelines, which include infrastructure costs like cloud services, storage, and network bandwidth.
Indirect Costs: Engineer time spent on debugging, fixing issues in production, and manual testing. This includes lost productivity due to context switching between tasks.
Calculate Opportunity Cost:
Value of Time: Estimate the value of an engineer’s time by calculating their hourly rate or using a standard industry figure (e.g., $100/hour).
Efficiency Improvement: Determine how much faster the feedback loop can be made and estimate the savings from reduced debugging and testing.
Trade-Off Analysis:
Use a cost-benefit analysis to compare the direct costs of CI/CD resources against the indirect cost savings from improved productivity.
Present Findings: Create a compelling business case that outlines how faster feedback loops can lead to better quality, lower risk, and higher efficiency.

#### Example
Model

Let's consider an example:

Current MTTD: 3 days (i.e., it takes on average three days for issues to be identified).
MTTR: 2 days (i.e., once identified, it takes two more days to resolve).
Average engineer cost per hour: $100.
Number of engineers affected: 50.

Calculation:

Daily Impact:

MTTD + MTTR = 3 + 2 = 5 days
Daily loss in productivity for each engineer = (MTTD + MTTR) * Engineer’s hourly rate / 8 hours/day [ \text{Daily Loss} = 5 \times $100 / 8 = $62.5 ]
Total daily loss for 50 engineers: [ \text{Total Daily Loss} = 50 \times $62.5 = $3,125 ]

Annual Impact:

Annual loss in productivity (assuming 250 working days): [ \text{Annual Loss} = $3,125 \times 250 = $781,250 ]

Direct Cost of CI/CD Resources: Assuming a modest increase in cloud infrastructure costs by $10,000 per month:

Annual cost: ( $10,000 \times 12 = $120,000 )

Net Savings:

Net savings from faster feedback loop: [ \text{Net Savings} = $781,250 - $120,000 = $661,250 ]

#### Closing Summary

By quantifying the economic impact of a slow CI/CD feedback loop in terms of lost productivity and direct costs, we can present a strong business case for optimizing this process. The model shows significant potential savings that outweigh modest increases in compute spend.

#### Common Mistakes/Follow-Ups

Overlooking indirect costs like engineer time.
Failing to justify the cost of CI/CD resources effectively.
Not considering long-term benefits such as improved quality and reduced risk.

**Comparison:**

```
Metric	Value
MTTD	3 days
MTTR	2 days
Hourly Rate	$100/hour
Num Engineers	50
Annual CI Cost	$120,000
```

**Code:**

```python
# Example Python script for calculation
def calculate_savings(mtt_d, mtr, hourly_rate, num_engineers):
    daily_loss = (mtt_d + mtr) * hourly_rate / 8
    total_daily_loss = daily_loss * num_engineers
    annual_loss = total_daily_loss * 250
    annual_ci_cost = 10000 * 12
    net_savings = annual_loss - annual_ci_cost
    return net_savings

mttd = 3
mtr = 2
hourly_rate = 100
num_engineers = 50

net_savings = calculate_savings(mttd, mtr, hourly_rate, num_engineers)
print(f"Net savings: ${net_savings}")
```

**Example input / output:**

```text
Input:
- MTTD = 3 days
- MTR = 2 days
- Hourly Rate = $100/hour
- Number of Engineers = 50

Output:
- Net savings from faster feedback loop = $661,250 per year
```

**Data:**

```json
{
  "mttd": 3,
  "mtr": 2,
  "hourlyRate": 100,
  "numEngineers": 50,
  "annualCost": 120000
}
```

**Flow:**

1. Define Metrics
2. MTTD + MTTR
3. Calculate Daily Loss
4. Total Daily Loss
5. Annual Loss
6. Net Savings
7. Presentation to Leadership

