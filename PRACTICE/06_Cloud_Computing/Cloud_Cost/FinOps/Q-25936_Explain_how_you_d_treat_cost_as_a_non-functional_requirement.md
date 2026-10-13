---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd treat cost as a non-functional requirement (like latency or availability) in system design docs, including how you'd set a cost SLO, budget the design against it, and reject designs that blow the budget. Give a concrete example.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost as a Non-functional Requirement](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_as_a_Non-functional_Requirement.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In the context of backend architecture as an architect or principal engineer, treating cost as a non-functional requirement (NFR) is crucial for ensuring financial sustainability and long-term viability of projects. This involves setting Service Level Objectives (SLOs) around cost, budgeting designs against these objectives, and rejecting those that exceed them. The key idea here is to balance the trade-offs between technical feasibility, performance, and cost while maintaining financial discipline.

#### Triaging the Question

The question asks how to handle cost as a non-functional requirement in system design documents. The core elements include:

Defining and setting SLOs for cost.
Budgeting the design against these objectives.
Rejecting designs that exceed the budget constraints.
Providing an example of a concrete scenario.

#### First Principles Solution

Defining Cost SLOs:

Establish baseline costs based on historical data or industry standards.
Define what constitutes acceptable cost variance (e.g., ±20% from expected).
Set specific, measurable, achievable, relevant, and time-bound (SMART) cost targets for different tiers of the service.

Budgeting Against Design:

Evaluate initial estimates based on cloud provider pricing models.

#### Cons
ider variable costs like compute, storage, network, and database usage.
Factor in fixed costs such as reserved instances, managed services, and licensing fees.
Use cost estimation tools provided by cloud providers or third-party services to refine budgets.

Design Review Process:

Integrate cost considerations into the design review process.
Ensure that each architectural decision is validated against the established SLOs.
Prioritize designs that offer both performance and cost efficiency.
Use tools like AWS Budgets, Azure Cost Management, or Google Cloud Cost Transparency to monitor and manage costs during implementation.

Rejecting High-Cost Designs:

Implement a gate review mechanism where high-cost proposals are flagged for further scrutiny.
Provide alternatives that meet the functional requirements within budget constraints.
Document the rationale for rejecting designs that exceed cost SLOs, including potential financial implications and risk mitigation strategies.
Concrete Example

Let's consider a microservices-based application serving a web service with heavy read/write operations. The key components are:

A front-end API gateway
Multiple backend services (database, caching, batch processing)
Monitoring and logging systems
Step 1: Defining Cost SLOs

Set an initial budget of $500 per month for the entire application.

Database costs (30%): Maximize use of read replicas and minimize provisioned storage.
Compute costs (40%): Optimize instance types based on CPU/Memory usage patterns.
Network costs (20%): Minimize egress traffic by caching frequently accessed data in a local cache.
Storage costs (10%): Use cost-effective storage options like S3 for non-critical backups.
Step 2: Budgeting Against Design

Assume the following initial design estimates:

Database (Amazon RDS): $75/month
Compute (EC2 t2.micro instances): $180/month
Network (VPC Peering): $40/month
Storage (S3 Standard): $20/month

Total estimated cost: $315/month, which is within the budget.

Step 3: Design Review Process

During the design review:

Validate that each microservice uses the most cost-effective instance types.
Ensure read/write operations are optimized to reduce database costs.
Implement caching strategies to minimize network and storage expenses.
Step 4: Rejecting High-Cost Designs

If a proposed change, such as adding an expensive managed service like AWS AppSync, would result in a total monthly cost of $600 or more:

Propose alternatives that meet the same functional requirements but are more cost-effective.
Document the trade-offs and potential financial implications.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, treating cost as an NFR involves setting clear SLOs, budgeting designs against them, and rejecting high-cost proposals. A concrete example illustrates the process of defining SMART cost targets, evaluating initial estimates, incorporating cost into design reviews, and ensuring financial discipline.

#### Common mistakes
include:

Overlooking variable costs like network egress.
Failing to document the rationale for rejecting designs that exceed budget constraints.
Not using tools effectively to monitor and manage cloud costs during implementation.

#### Follow-ups
might include discussing specific tools or services used, additional cost optimization techniques, or how to handle unexpected spikes in usage.

**Comparison:**

```
Front-end API Gateway
Database
Backend Services
Caching
Batch Processing
Networking
Monitoring & Logging
Storage
Optimize Read/WriteOperations
Implement Local Cache
Minimize Network Traffic
Use Cost-Effective Storage
```

**Code:**

```yaml
# Example YAML for AWS Budgets
Resources:
  - Type: "AWS::Budgets::Budget"
    Properties:
      Budget:
        TimePeriod:
          Start: <current_date>
          End: <one_year_from_now>
        BudgetType: "COST"
        BudgetLimit:
          Amount: 500.00
          Unit: "USD"
        Alarms:
          - AlarmName: "CostExceededAlarm"
            ComparisonOperator: LESS_THAN_OR_EQUAL
```

**Example input / output:**

| Component | Initial Estimate (USD/month) | Optimized Cost (USD/month) |
|-----------|-----------------------------|----------------------------|
| Database  | $75                         | $60                        |
| Compute   | $180                        | $130                       |
| Network   | $40                         | $20                        |
| Storage   | $20                         | $15                        |
| **Total** | **$315**                    | **$225**                   |

**Data:**

```json
{
  "database": {
    "type": "RDS",
    "cost_per_month": 75.00,
    "optimizations": [
      "Use read replicas",
      "Minimize provisioned storage"
    ]
  },
  "compute": {
    "type": "EC2 t2.micro instances",
    "cost_per_month": 180.00,
    "optimizations": [
      "Optimize instance types based on usage patterns",
      "Use spot instances when possible"
    ]
  },
  "network": {
    "type": "VPC Peering",
    "cost_per_month": 40.00,
    "optimizations": [
      "Minimize egress traffic by caching data in local storage"
    ]
  },
  "storage": {
    "type": "S3 Standard",
    "cost_per_month": 20.00,
    "optimizations": [
      "Use cost-effective storage options for non-critical backups"
    ]
  }
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. Start
4. Start --> DefineCostSLOs
5. DefineCostSLOs --> EstimateInitialCosts
6. EstimateInitialCosts --> EvaluateDesignAgainstBudget
7. EvaluateDesignAgainstBudget --> ReviewCostOptimizations
8. ReviewCostOptimizations --> ValidateProposedChanges
9. ValidateProposedChanges --> DocumentRationale
10. DocumentRationale --> End
11. subgraph DefineCostSLOs
12. DefineCostSLOs --> SetBaselineCost
13. SetBaselineCost --> EstablishSMARTTargets
14. end
15. subgraph EstimateInitialCosts
16. EstimateInitialCosts --> DatabaseCostEstimate
17. DatabaseCostEstimate --> ComputeCostEstimate
18. ComputeCostEstimate --> NetworkCostEstimate
19. NetworkCostEstimate --> StorageCostEstimate
20. StorageCostEstimate --> TotalEstimatedCost
21. end
22. subgraph EvaluateDesignAgainstBudget
23. EvaluateDesignAgainstBudget --> GateReviewMechanism
24. GateReviewMechanism --> RejectedHighCostProposals
25. RejectedHighCostProposals --> ProvideAlternatives
26. ProvideAlternatives --> DocumentTradeoffs
27. end
28. subgraph ReviewCostOptimizations
29. ReviewCostOptimizations --> ImplementStrategies
30. ImplementStrategies --> MonitorAndManageCosts
31. end

