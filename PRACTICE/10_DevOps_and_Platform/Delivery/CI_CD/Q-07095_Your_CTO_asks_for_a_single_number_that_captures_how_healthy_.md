---
type: "practice"
difficulty: "Deep"
---

# Your CTO asks for a single number that captures 'how healthy is our delivery'; you know one number is dangerous. How do you push back, what balanced scorecard do you propose instead, and how do you prevent it from being weaponized in perf reviews?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ CI Cost Optimization](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_Cost_Optimization/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When presented with a single "health" metric to capture the state of our CI/CD pipeline, I would push back by proposing a balanced scorecard approach that provides a more nuanced and comprehensive view of our delivery health. A single number can be misleading or misinterpreted, leading to skewed priorities in performance reviews. Instead, we need a multi-dimensional assessment that covers various aspects of our software development lifecycle.

Firstly, let's break down the core components of CI/CD health:

Build Success Rate: Measures how often builds succeed.
Test Coverage and Failures: Tracks code coverage and integration test failures.
Deployment Frequency: How frequently we deploy new features or fixes to production.
Lead Time for Changes: The time it takes from making a change in source control to its availability in production.
Change Failure Rate: The percentage of changes that fail on deployment.

A balanced scorecard approach would aggregate these metrics into a more holistic view, ensuring no single dimension is overlooked. This allows us to identify bottlenecks and areas for improvement without putting undue pressure on any one metric.

#### Triaging the Question

The CTO's request for a single "health" number reflects a common misconception that simplification always leads to better decision-making. In reality, complex systems benefit from a multifaceted assessment. The key is to ensure that our metrics support strategic goals while providing actionable insights.

#### First-Principles Solution

Define the Metrics: Start by defining clear and meaningful metrics for each component of CI/CD.

Build Success Rate (BSR): Measure as a percentage of successful builds out of all attempted builds. [ BSR = \frac{\text{Number of Successful Builds}}{\text{Total Number of Builds}} ]
Test Coverage and Failures (TCF): Track both code coverage and integration test failures to ensure robust testing.
Code Coverage: [ \text{Code Coverage} = \left( \frac{\text{Number of Executed Lines}}{\text{Total Number of Lines in Source Code}} \right) \times 100 ]
Integration Test Failures: Count the number of integration test failures per build.
Deployment Frequency (DF): Measure how often deployments occur to production. [ DF = \frac{\text{Number of Deployments}}{\text{Time Period}} ]
Lead Time for Changes (LTC): Calculate as the average time from code commit to deployment in production. [ LTC = \left( \sum_{i=1}^{n} (\text{Commit}{i} - \text{Deployment}{i}) \right) / n ]
Change Failure Rate (CFR): Measure the percentage of changes that fail on deployment. [ CFR = \frac{\text{Number of Failed Changes}}{\text{Total Number of Changes}} ]

Weight and Combine Metrics: Combine these metrics into a balanced scorecard, assigning appropriate weights based on their importance to our business objectives. [ CI/CD Health Score = w_1 \times BSR + w_2 \times TCF + w_3 \times DF + w_4 \times LTC + w_5 \times CFR ] Where (w_i) are the weights assigned to each metric.

#### Closing Summary
and Common Mistakes/Follow-Ups

By using a balanced scorecard, we can provide a more accurate picture of our CI/CD health. This approach ensures that all critical aspects are considered, reducing the risk of overemphasizing one dimension at the expense of others. It also prevents the single number from being weaponized in performance reviews by providing a holistic view.

#### Common mistakes
include:

Overreliance on a Single Metric: Ignoring other important factors.
Ignoring Outliers and Anomalies: Failing to address issues that can significantly impact overall health.
Misaligned Weights: Assigning incorrect weights, leading to biased assessments.

To prevent these issues, it's crucial to regularly review the scorecard, update metrics as needed, and ensure alignment with business goals. Additionally, fostering a culture of transparency and collaboration will help mitigate the risk of performance reviews being influenced by a single metric.

**Comparison:**

```
Metric	Definition
Build Success Rate	Percentage of successful builds out of all attempted builds
Test Coverage	Proportion of source code that is covered by automated tests
Deployment Frequency	Number of deployments to production per time period
Lead Time for Changes	Average time from commit to deployment in production
Change Failure Rate	Percentage of changes that fail on deployment
```

**Code:**

```python
# Example Python code for calculating some of these metrics
def build_success_rate(builds):
    return (builds['successful'] / builds['total']) * 100

def lead_time_for_changes(changes):
    commit_dates = [change['commit_date'] for change in changes]
    deployment_dates = [change['deployment_date'] for change in changes]
    times = [(deploy - commit).days for deploy, commit in zip(deployment_dates, commit_dates)]
    return sum(times) / len(times)

# Example data
builds_data = {'successful': 120, 'total': 150}
changes_data = [{'commit_date': datetime(2023, 9, 1), 'deployment_date': datetime(2023, 9, 3)},
                {'commit_date': datetime(2023, 9, 4), 'deployment_date': datetime(2023, 9, 6)}]

print("Build Success Rate:", build_success_rate(builds_data))
print("Lead Time for Changes:", lead_time_for_changes(changes_data))
```

**Example input / output:**

```text
Build Success Rate: 80.0%
Lead Time for Changes: 1.5 days
```

**Data:**

```json
{
  "builds": {
    "successful": 120,
    "total": 150
  },
  "changes": [
    {
      "commit_date": "2023-09-01",
      "deployment_date": "2023-09-03"
    },
    {
      "commit_date": "2023-09-04",
      "deployment_date": "2023-09-06"
    }
  ]
}
```

**Flow:**

1. Define Metrics
2. Weight and Combine
3. Calculate Scores
4. Update Scorecard
5. Review and Adjust Weights

