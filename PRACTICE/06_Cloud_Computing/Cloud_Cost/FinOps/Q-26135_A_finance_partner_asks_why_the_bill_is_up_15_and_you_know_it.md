---
type: "practice"
difficulty: "Intermediate"
---

# A finance partner asks why the bill is up 15% and you know it's healthy revenue-driven growth. How would you present cost alongside a business metric so it doesn't look like a problem?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Healthy vs Wasteful Cost Growth](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Healthy_vs_Wasteful_Cost_Growth.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To address the concern of an 15% increase in cloud costs, we'll present a comprehensive overview that aligns cost metrics with key business metrics such as revenue growth. This approach ensures that the cost increase is seen as a result of healthy business activity rather than a financial issue.

#### Triaging the Question

The finance partner's concern is about the 15% increase in cloud costs, which they suspect might not be due to healthy revenue-driven growth. The goal is to present this data in a way that highlights the correlation between cost and business performance metrics like revenue or customer acquisition rate (CAC). This requires us to:

Understand the baseline cost structure.
Identify key business drivers causing the increase.
Calculate relevant cost-to-revenue ratios.
Visualize the data effectively.

#### First Principles Solution
Step 1: Gather Baseline Data

First, collect historical cloud cost and revenue data for the past several months. This will serve as a reference point to understand normal variations in costs.

PLAINTEXT
| Month | Revenue ($) | Cloud Cost ($) |
|-------|-------------|---------------|
| Jan   | 500,000     | 40,000        |
| Feb   | 520,000     | 41,000        |
| Mar   | 550,000     | 43,000        |
| Apr   | 600,000     | 50,000        |
Step 2: Identify Key Business Drivers

Analyze the business performance metrics to identify factors that might drive up costs. For instance:

Revenue growth: Increased customer acquisition.
Productivity enhancements: More frequent usage of cloud services.
Step 3: Calculate Cost-to-Revenue Ratios

Compute the cost as a percentage of revenue for each month:

PLAINTEXT
| Month | Cost/Revenue Ratio |
|-------|--------------------|
| Jan   | 8%                 |
| Feb   | 7.9%               |
| Mar   | 7.8%               |
| Apr   | 8.3%               |
Step 4: Visualize the Data

Create charts and graphs to show the relationship between revenue growth and cloud costs over time.

bar
    title Cloud Costs vs Revenue Growth
    xaxis Year-Month
    yaxis [Revenue, Cost]
    "2023-01": [500000, 40000]
    "2023-02": [520000, 41000]
    "2023-03": [550000, 43000]
    "2023-04": [600000, 50000]
Step 5: Present the Findings

In a meeting or report, present the data and highlight:

The overall growth in revenue.
How cloud costs have increased proportionally to this growth.
Any specific initiatives that contributed to both cost and revenue.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Presenting cost alongside business metrics effectively communicates that an increase is justified by healthy business performance. Key takeaways:

Cost should be seen as a necessary investment for growth, not an independent issue.
Regularly monitor and adjust cloud spend based on actual usage and revenue trends.

#### Common mistakes
include:

Failing to correlate costs with business drivers.
Overlooking historical data, leading to misinterpretation of current cost increases.
Not visualizing the data clearly, making it harder for stakeholders to understand the relationship between metrics.

#### Follow-ups
might involve setting up automated monitoring tools or implementing more granular financial controls to ensure future growth is sustainable and well-managed.

**Comparison:**

```
Month	Revenue ($)	Cloud Cost ($)
Jan	500,000	40,000
Feb	520,000	41,000
Mar	550,000	43,000
Apr	600,000	50,000
```

**Code:**

```python
# Example Python code for calculating cost/revenue ratio
revenue_data = [500000, 520000, 550000, 600000]
cost_data = [40000, 41000, 43000, 50000]

ratios = [(cost / revenue) * 100 for cost, revenue in zip(cost_data, revenue_data)]
print(ratios)
```

**Example input / output:**

```text
Input:
- Historical revenue and cloud cost data

Output:
- Cost/revenue ratios for each month
- Visualized charts comparing revenue growth and cloud costs
```

**Data:**

| Month | Revenue ($) | Cloud Cost ($) |
|-------|-------------|---------------|
| Jan   | 500,000     | 40,000        |
| Feb   | 520,000     | 41,000        |
| Mar   | 550,000     | 43,000        |
| Apr   | 600,000     | 50,000        |

**Flow:**

1. Collect Baseline Data
2. Identify Business Drivers
3. Calculate Cost/Revenue Ratios
4. Visualize the Data
5. Present Findings

