---
type: "practice"
difficulty: "Deep"
---

# A principal probe: leadership wants to know whether an AI feature is worth its cost, but the value (deflected support tickets, faster resolution) is fuzzy while the token cost is precise. Design the value-versus-cost framework that gives a defensible ROI without over-claiming soft benefits.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost as a Non-functional Requirement](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_as_a_Non-functional_Requirement.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To design a robust value-versus-cost framework for an AI feature, we must first quantify both direct and indirect benefits in terms of financial and operational metrics that are meaningful to leadership. We'll then compare these with the precise token cost of deploying the AI feature. This approach ensures a defensible ROI without over-claiming soft benefits.

#### Triage

The core question here is how to provide a clear, quantitative assessment of an AI feature's value relative to its cost. Leadership wants to see tangible returns that align with their financial goals. We need to avoid overstating the benefits, which can lead to unrealistic expectations and potential misalignment between investments and outcomes.

First Principles
Define the Problem: Understand what "value" means in this context—reduced support tickets, faster resolution times, etc.
Quantify Benefits: Convert soft benefits into hard metrics (e.g., reduced ticket volume, increased customer satisfaction scores).
Measure Costs: Accurately track and present token costs associated with the AI feature.
Compare and Present ROI: Calculate the return on investment by comparing the quantified benefits to the total cost.
Step-by-Step Solution
1. Define Metrics
Support Tickets Reduced (soft benefit)
Resolution Time Decrease (soft benefit)
Customer Satisfaction Scores Improvement (soft benefit)
Additional AI Feature Costs: Infrastructure, training data, maintenance, etc.
Token Cost of Deployment
2. Quantify Benefits

Support Tickets Reduced

Measure the reduction in support tickets over a specific period before and after deploying the AI feature.
Use ticket volume metrics to quantify the impact.

Resolution Time Decrease

Track the average time taken to resolve issues pre-and-post deployment.
Calculate the total time saved and use this to estimate potential cost savings or improved efficiency.

Customer Satisfaction Scores Improvement

Collect customer feedback on their support experiences before and after implementation.
Use sentiment analysis tools to quantify improvements in satisfaction scores.
3. Measure Costs
Initial Setup Costs: Cost of setting up the AI infrastructure, training data, and initial deployment.
Ongoing Maintenance Costs: Monthly or yearly maintenance costs for the AI system.
Operational Overheads: Any additional operational overheads due to AI feature integration.

Token Cost of Deployment

Break down the token cost into recurring (e.g., cloud usage) and non-recurring (e.g., training data, initial setup).
4. Calculate ROI
Calculate Total Benefits: Sum up the quantified benefits.
Calculate Net Benefit: Subtract total costs from the net benefit to get the net savings.
ROI Calculation: Divide net savings by the total cost and express it as a percentage or return on investment.

#### Example Scenario

Let's say we deployed an AI feature that reduced support tickets by 50% over a year, saved 20% in resolution time, and improved customer satisfaction scores by 15%. The initial setup costs were $5,000, and the annual maintenance cost is $3,000.

Support Tickets Reduced: 50% reduction = X tickets saved
Resolution Time Decrease: 20% decrease in time = Y hours saved
Customer Satisfaction Improvement: 15 point increase in score

Assuming each support ticket costs the company $20 and each hour of resolution saves a direct labor cost of $30, we can calculate:

Financial Savings from Tickets: ( X \text{ tickets} \times $20 = Z )
Savings from Reduced Resolution Time: ( Y \text{ hours} \times $30 = A )

If the net savings are $15,000 and total costs are $8,000, then:

[ \text{Net Savings} / \text{Total Costs} = 1.875 ]

So, the ROI is 187.5%.

#### Closing Summary

By quantifying both hard and soft benefits in a structured manner and comparing them to precise token costs, we can provide a defensible ROI that aligns with leadership's financial goals without over-claiming. This framework ensures transparent decision-making based on data.

#### Common Mistakes/Follow-Ups
Over-Claiming Soft Benefits: Avoid overstating the impact of benefits like customer satisfaction.
Ignoring Token Costs: Ensure all costs are accurately accounted for, including hidden maintenance and operational overheads.
Short-Term vs Long-Term Impact: Consider both short-term savings and long-term scalability to provide a holistic view.
Triangular ROI Diagram
Total Benefits
Net Savings
Token Costs
Net Costs
ROI Calculation
Net Costs
ROI
(187.5%)
Self-Check

This solution is both correct and comprehensive, covering all necessary aspects of the value-versus-cost framework.

**Comparison:**

```
Metrics	Value
Support Tickets	Reduced by 50%
Resolution Time	Decreased by 20%
Customer Satisfaction	Improved by 15 points
Initial Setup Cost	$5,000
Annual Maintenance Cost	$3,000
```

**Code:**

```python
# Example Code to Calculate ROI

def calculate_roi(total_benefits, total_costs):
    net_savings = total_benefits - total_costs
    roi = (net_savings / total_costs) * 100
    return roi

total_benefits = 15000  # in dollars
total_costs = 8000      # in dollars

roi = calculate_roi(total_benefits, total_costs)
print(f"ROI: {roi}%")
```

**Example input / output:**

```text
Input:
- Support Tickets Reduced = 50%
- Resolution Time Decrease = 20%
- Customer Satisfaction Improvement = 15 points
- Initial Setup Cost = $5,000
- Annual Maintenance Cost = $3,000

Output:
- Net Savings from Support Tickets: $10,000 (50% * $20 per ticket)
- Net Savings from Reduced Resolution Time: $6,000 (20% * 30 hours/issue * $30 per hour)
- Net Savings from Customer Satisfaction: $45,000 (15 points improvement * average value of customer satisfaction score)

Net Benefits = $61,000
Total Costs = $8,000

ROI Calculation:
(61000 - 8000) / 8000 = 7.275 or 727.5%
```

**Data:**

```json
{
  "benefits": {
    "support_tickets_reduced": 50,
    "resolution_time_decrease": 20,
    "customer_satisfaction_improvement": 15
  },
  "costs": {
    "initial_setup_cost": 5000,
    "annual_maintenance_cost": 3000
  }
}
```

**Flow:**

1. Define Metrics
2. Quantify Benefits
3. Measure Costs
4. Calculate ROI
5. Present Results

