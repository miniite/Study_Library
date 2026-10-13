---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd embed cost as a non-functional requirement in your architecture-decision-record (ADR) process, so every significant design carries a steady-state and peak cost estimate and a cost-budget the design is held to. What forces the estimate to be honest, and how do you audit ADRs against actuals a year later?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost as a Non-functional Requirement](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_as_a_Non-functional_Requirement.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an architecture-decision-record (ADR) process within a DevOps context, embedding cost as a non-functional requirement ensures that every significant design carries accurate steady-state and peak cost estimates, which are then rigorously enforced through budgetary constraints. To achieve this, we need to create a robust ADR template that includes detailed cost calculations and ongoing audit mechanisms.

Firstly, let’s clarify what the question is asking: The goal is to integrate cost considerations into our design decisions at an early stage of development. This involves not only estimating costs but also ensuring these estimates are realistic and enforceable through budgetary constraints defined in ADRs. We need a system that forces honesty in cost estimation and allows us to audit those estimates against actual performance one year later.

To achieve this, we will follow the steps outlined below:

Define the Cost-Embedded ADR Template: The ADR template should include sections for steady-state and peak costs, justification for these costs, and a budgetary constraint that the design must adhere to.
Cost Estimation Process: This involves detailed analysis of resource requirements, usage patterns, and expected scale. We use tools like AWS Budgets, Azure Cost Management, or Google Cloud Pricing Calculator to provide accurate cost estimates.
Honesty Mechanisms: We introduce two key mechanisms: (a) a peer review process that ensures multiple perspectives are considered, and (b) a data-driven approach where historical usage data is used to validate the accuracy of the initial estimate.
Audit Process: One year later, we audit the actual costs against the estimated costs. This involves comparing the budgeted versus actual spend and identifying any discrepancies.

Now, let’s break this down step-by-step:

1. Define the Cost-Embedded ADR Template

Template Example:

CODE
# Title: [Project Name] - Architecture Decision Record
## Background
[Description of project and its objectives]

## Options Considered
[A list of architectural options considered]
   - Option A
      - Pros:
         - ...
      - Cons:
         - ...
      - Estimated Cost (Steady-state): $XXX
      - Estimated Cost (Peak): $YYY

   - Option B
      - ...

### Recommended Option: [Option X]
Justification for selection:

## Implementation Strategy
- Key steps to implement the chosen option
- Budgetary constraint for steady-state and peak costs:
  - Steady-State: $ZZZ
  - Peak: $AAA

## Risks and Mitigation
- Potential risks related to cost overruns, and mitigation strategies.
2. Cost Estimation Process

We use a combination of historical data, predictive models, and real-time monitoring tools:

Historical Data
Usage Patterns
Predictive Models
Cost Calculations
Real-Time Monitoring Tools
3. Honesty Mechanisms

To ensure honesty, we introduce a peer review process where the team members involved in the cost estimation provide their inputs and justifications. This is supplemented by using data-driven validation:

Historical Data Validation: Use past projects to validate the accuracy of current cost estimates.
Automated Alerts: Set up alerts for deviations from expected costs.
4. Audit Process

One year later, we perform a detailed audit against actual costs:

MARKDOWN
| Metric          | Expected (Steady-State) | Actual |
|-----------------|------------------------|--------|
| Compute Usage   | $500                   | $600   |
| Storage Costs   | $200                   | $180   |
| Network Costs   | $300                   | $400   |

**Total Expected: $1,000**
**Actual Total: $1,200**

Discrepancy Analysis:
- Compute Usage: +$100
- Storage Costs: -$20
- Network Costs: +$100

Possible Causes:
- Increased demand for compute resources due to unanticipated growth.
- Decreased storage costs due to optimization efforts.
- Unforeseen network usage during peak hours.

Recommendations:
- Review and adjust the budget constraints.
- Implement cost-saving measures where feasible.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key is to ensure that cost considerations are an integral part of every architecture decision. By embedding costs in our ADR process, we can make informed decisions upfront that align with financial realities. However, common pitfalls include:

Lack of Detailed Estimations: Not having thorough and well-informed cost estimates.
Inadequate Peer Reviews: Relying solely on one person’s estimate without cross-verifying.
Insufficient Audit Mechanisms: Failing to set up effective mechanisms for comparing actual costs against estimated ones.

To avoid these issues, it is crucial to have a structured process and robust tools in place. This ensures that cost considerations are not just a box-ticking exercise but a fundamental part of our architecture decision-making.

**Comparison:**

| Metric          | Expected (Steady-State) | Actual |
|-----------------|------------------------|--------|
| Compute Usage   | $500                   | $600   |
| Storage Costs   | $200                   | $180   |
| Network Costs   | $300                   | $400   |

**Code:**

```python
# Example script for generating ADR templates
def generate_adr_template(title, project_name):
    template = f"### Title: {title} - Architecture Decision Record\n## Background\n[Description of project and its objectives]\n\n## Options Considered\n[A list of architectural options considered]"
    return template

template = generate_adr_template("Project X", "Cloud Cost Governance")
print(template)
```

**Example input / output:**

```text
**Input:**
- Steady-State Costs:
  - Compute Usage: $500
  - Storage Costs: $200
  - Network Costs: $300

- Peak Costs:
  - Compute Usage: $600
  - Storage Costs: $180
  - Network Costs: $400

**Output:**
| Metric          | Expected (Steady-State) | Actual |
|-----------------|------------------------|--------|
| Compute Usage   | $500                   | $600   |
| Storage Costs   | $200                   | $180   |
| Network Costs   | $300                   | $400   |

**Discrepancy Analysis:**
- Compute Usage: +$100
- Storage Costs: -$20
- Network Costs: +$100

**Recommendations:**
- Review and adjust the budget constraints.
- Implement cost-saving measures where feasible.
```

**Data:**

```json
{
  "SteadyStateCost": {
    "ComputeUsage": 500,
    "StorageCosts": 200,
    "NetworkCosts": 300
  },
  "PeakCost": {
    "ComputeUsage": 600,
    "StorageCosts": 180,
    "NetworkCosts": 400
  }
}
```

**Flow:**

1. Initial Estimate
2. Peer Review
3. Predictive Models
4. Cost Calculations
5. Audit Against Actuals
6. Budgetary Constraints

