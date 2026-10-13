---
type: "practice"
difficulty: "Deep"
---

# You are evaluating a GenAI platform where the team has already spent nine months building a bespoke in-house RAG-and-agent framework. How do you run an ATAM that surfaces the real build-vs-buy risks without triggering sunk-cost defensiveness that turns the review into theatre?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Regulatory Requirements in Utility Trees](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Regulatory_Requirements_in_Utility_Trees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating whether to continue building a bespoke in-house RAG-and-agent framework or switching to a commercial solution, it's crucial to conduct an Architectural Trade-off Analysis (ATAM) that is both thorough and unbiased. ATAM involves systematically identifying and comparing the trade-offs across various dimensions like cost, performance, maintainability, security, and compliance. The key challenge here is to avoid sunk-cost bias and ensure the review process remains objective and constructive.

The first step in running this analysis is to clearly define the goals of the platform. For instance, we need to determine the desired level of accuracy, response time, and integration with existing systems. This helps us to set a baseline for comparing different options against these metrics.

Next, we should establish a list of criteria that will be used to evaluate both building vs buying scenarios. These criteria could include:

Cost: Initial development costs versus ongoing maintenance and support.
Performance: Response times and scalability under various loads.
Maintainability: Ease of updates and dependencies on third-party services or libraries.
Security: Compliance with data protection regulations and security standards.
Compliance: Regulatory compliance requirements and audit trails.

We will then create a structured framework to evaluate both options. For the in-house solution, we would consider:

The current state of development: Is it feature-complete? What are the known bugs or limitations?
Development team size and skill set: Are there enough experts to continue development and support?
Integration with existing systems: How complex will integration be?

For a commercial solution, we need to evaluate:

Licensing costs: One-time fees vs. recurring subscription models.
Vendor reliability: History of service uptime, support quality, and innovation pace.
Customizability: Can the platform be extended or customized to meet specific needs?
Integration effort: How easy is it to integrate with existing systems?

To avoid sunk-cost bias, we should treat this as a fresh start. We can use techniques like "pulling the trigger" to ensure that our decision-making process is objective and not influenced by past investments. This involves pretending that no work has been done so far and evaluating each option independently.

Next, we will perform a detailed cost-benefit analysis for both options. For example:

In-house framework: Calculate man-hours spent on development, maintenance, and potential savings from reduced licensing costs.
Commercial solution: Estimate one-time setup costs versus recurring subscription fees.

We should also consider long-term strategic implications such as vendor lock-in, technological obsolescence, and the need for future scalability. For instance, if a commercial platform has a strong track record of innovation, it might offer better long-term value despite higher initial costs.

Finally, we will document our findings in a detailed report that outlines the trade-offs clearly. We should present both qualitative assessments (e.g., security risks) and quantitative data (e.g., cost comparisons). The goal is to make an informed decision based on facts rather than emotions or historical investments.

**Code:**

```python
# Example of a simplified cost-benefit analysis function
def evaluate_option(option_data):
    total_cost = option_data['initial_cost'] + option_data['maintenance_cost']
    value = option_data['revenue'] - total_cost
    return value

# Data for in-house framework and commercial solution
in_house = {
    'initial_cost': 150000,
    'maintenance_cost': 60000,
    'revenue': 300000
}

commercial_solution = {
    'license_cost': 20000,
    'subscription_cost': 15000,
    'revenue': 300000
}
```

**Example input / output:**

| Option             | Initial Cost | Maintenance Cost | Total Cost | Revenue |
|--------------------|--------------|------------------|------------|---------|
| In-House Framework | $150,000     | $60,000          | $210,000   | $300,000|
| Commercial Solution| $20,000      | $15,000          | $35,000    | $300,000|

**Data:**

```json
{
  "in_house": {
    "initial_cost": 150000,
    "maintenance_cost": 60000,
    "revenue": 300000
  },
  "commercial_solution": {
    "license_cost": 20000,
    "subscription_cost": 15000,
    "revenue": 300000
  }
}
```

