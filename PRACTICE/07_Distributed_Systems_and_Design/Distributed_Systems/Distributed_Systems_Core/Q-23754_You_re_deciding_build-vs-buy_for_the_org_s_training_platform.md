---
type: "practice"
difficulty: "Expert"
---

# You're deciding build-vs-buy for the org's training platform: run your own Kubernetes/Ray/Slurm-based stack on owned GPUs versus a managed training service. Frame it on cost at scale, GPU availability and lock-in, operational load of fault-tolerant training, and where the crossover point sits for a company scaling ML.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Choosing an Event Backbone](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Choosing_an_Event_Backbone.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating whether to build or buy a training platform based on Kubernetes/Ray/Slurm versus a managed service, we must consider several key factors at scale: cost, GPU availability and lock-in, and operational load of maintaining fault-tolerant training environments. The core question is where the crossover point lies for the organization's scaling needs.

#### Triaging the Question

The primary goal here is to determine if building our own platform offers a better total cost of ownership (TCO) compared to using a managed service at different scales, while also assessing operational challenges such as GPU resource management and fault tolerance. The managed service typically comes with some level of lock-in but provides immediate scalability and reduced upfront costs.

#### Assumptions
Scale: The organization is scaling its ML workloads significantly.
GPU Usage: GPUs are a critical resource for training models, and their availability directly impacts TCO.
Operational Expertise: The team has the necessary expertise to manage Kubernetes/Ray/Slurm effectively.

#### First-Principles Solution

Cost Analysis

Build Option:

Initial Infrastructure Costs: Cost of purchasing or leasing GPUs, setting up and configuring infrastructure (hardware, network, storage).
Maintenance Costs: Ongoing costs for maintaining the hardware, software licenses (e.g., Kubernetes), and personnel.
Operational Costs: Time spent on managing clusters, monitoring resource usage, and handling failures.

Buy Option:

Subscription Costs: Monthly or yearly fees from a managed service provider.
Support Costs: Technical support and SLAs provided by the managed service.
Scalability: Managed services often offer automated scaling with minimal operational effort.

GPU Availability

Build Option:

Resource Allocation: Manual allocation of GPUs, potentially leading to underutilization or overprovisioning.
Fault Tolerance: Implementing robust fault-tolerant strategies (e.g., distributed training) can be complex and resource-intensive.

Buy Option:

Automatic Scaling: Managed services often provide automated scaling based on demand.
High Availability: Providers typically have built-in redundancy and failover mechanisms.

Lock-In

Build Option:

Vendor Lock-In: Using open-source tools like Kubernetes, Ray, or Slurm might reduce lock-in compared to proprietary services.
Customization: Greater flexibility in customizing the stack to specific needs.

Buy Option:

Provider Lock-In: Reliance on a specific managed service can limit future flexibility and vendor choice.

Operational Load

Build Option:

Complexity: Setting up and managing Kubernetes, Ray, or Slurm clusters requires significant effort.
Maintenance: Handling software updates, security patches, and troubleshooting issues.

Buy Option:

Simplified Operations: Reduced operational overhead due to managed services’ automated processes.
Crossover Point Analysis

To determine the crossover point where building outweighs buying or vice versa, we need to model these factors mathematically. Here’s a simplified approach:

Cost Model [ TCO_{\text{build}} = I + M + O ]

(I) is initial infrastructure cost.
(M) is maintenance and operational costs.
(O) is ongoing operational costs.

Managed Service Cost [ TCO_{\text{buy}} = S ]

(S) is the total subscription cost.

Scaling Factor Let’s assume a scaling factor (F) that represents the number of models or training jobs. As (F) increases, so do costs and operational complexity differently for build vs. buy.

Break-even Analysis Set up an equation where: [ TCO_{\text{build}}(F) = TCO_{\text{buy}} ] Solve for the value of (F) at which both options are equal.

#### Closing Summary

In summary, the decision between building or buying a training platform hinges on the specific scaling needs and operational expertise. For smaller organizations or startups, the managed service might offer better TCO due to reduced upfront costs and simplified operations. However, for larger enterprises with significant ML workloads and existing technical infrastructure, building a custom solution could provide greater control and cost savings over time.

#### Common Mistakes
Overestimating Build Costs: Failing to account for long-term maintenance and operational costs.
Underestimating Managed Service Costs: Not fully considering subscription fees and hidden costs.
Ignoring Operational Complexity: Underestimating the effort required to manage a custom solution, especially in terms of fault tolerance and resource management.

#### Conclusion

For an organization scaling ML at significant levels, building a custom training platform on Kubernetes/Ray/Slurm might offer better long-term cost benefits, but thorough analysis is essential. Carefully evaluate the costs, operational complexity, and potential lock-in to make an informed decision.

**Comparison:**

```
2021-01-01
2022-01-01
2023-01-01
2024-01-01
2025-01-01
2026-01-01
2027-01-01
Build_Initial
Buy_Subscription
Build_Maintenance
Buy_Support
Build_OpCosts
Buy_OpCosts
BreakEven
Initial Costs
Operational Costs
Maintenance
Cross-over Analysis
Crossover Point Analysis
```

**Code:**

```python
# Example Python code for simple TCO calculation
def calculate_tco_build(initial_cost, maintenance_cost, op_cost_per_model, models):
    total_op_cost = op_cost_per_model * models
    return initial_cost + maintenance_cost + total_op_cost

def calculate_tco_buy(subscription_cost_per_model, models):
    return subscription_cost_per_model * models

# Example parameters for break-even analysis
initial_infrastructure_cost = 50000  # Initial infrastructure cost in USD
maintenance_cost = 20000             # Maintenance and operational costs per year in USD
op_cost_per_model = 10               # Ongoing operational cost per model in USD
subscription_cost_per_model = 3      # Subscription cost per model in USD

models = range(1, 100)  # Example range of models from 1 to 99

# Calculate TCO for both options over the range of models
tco_build_values = [calculate_tco_build(initial_infrastructure_cost, maintenance_cost, op_cost_per_model, m) for m in models]
tco_buy_values = [calculate_tco_buy(subscription_cost_per_model, m) for m in models]

# Find the crossover point
crossover_point = next((i for i in range(len(tco_build_values)) if tco_build_values[i] > tco_buy_values[i]), None)
```

**Example input / output:**

```text
| Models | TCO_Build ($) | TCO_Buy ($) |
|--------|--------------|------------|
| 1      | 70,000       | 3          |
| 25     | 97,500       | 75         |
| 50     | 145,000      | 150        |
| 75     | 182,500      | 225        |
| 100    | 220,000      | 300        |

The table shows the TCO for both build and buy options as the number of models increases.
```

**Data:**

```json
{
  "initial_infrastructure_cost": 50000,
  "maintenance_cost": 20000,
  "op_cost_per_model": 10,
  "subscription_cost_per_model": 3
}
```

**Flow:**

1. Yes
2. No
3. Initial Infrastructure Cost
4. Maintenance & OperationalCosts
5. Models > X?
6. Build TCO < Buy TCO
7. Buy TCO < Build TCO

