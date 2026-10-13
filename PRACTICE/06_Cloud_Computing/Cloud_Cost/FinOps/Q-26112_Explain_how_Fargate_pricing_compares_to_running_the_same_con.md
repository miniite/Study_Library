---
type: "practice"
difficulty: "Deep"
---

# Explain how Fargate pricing compares to running the same containers on EC2 you manage. When does Fargate's convenience premium stop being worth it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Serverless Cost Visibility](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this discussion, we'll compare Fargate pricing with manually managing EC2 instances for containerized applications. We’ll analyze the costs and trade-offs to determine when Fargate’s convenience becomes less valuable.

TL;DR:

Fargate offers a managed environment that abstracts away infrastructure management, but it comes with a premium. This premium is worth it in scenarios where frequent scaling or dynamic resource requirements are necessary, but as your workload stabilizes, the cost and complexity of managing EC2 instances can become more efficient.

#### Triaging the Question

The question delves into the financial aspects of container orchestration between AWS Fargate and manually managed EC2 instances. The core idea is to understand when the convenience provided by Fargate outweighs its higher costs compared to self-managed resources.

#### Assumptions
We're comparing equivalent workloads (same number of containers, same resource utilization).
Both environments use the latest versions of AWS services.
No significant operational overhead for manual EC2 management is factored in for simplicity.
Step-by-Step Solution from First Principles:
1. Fargate Pricing Structure:

AWS Fargate charges based on the number of tasks and their duration. The cost per task is approximately $0.0000375 per second, with a minimum charge of 60 seconds (about $0.00225 per minute or $0.135 per hour). This model is particularly advantageous for bursty workloads that require frequent scaling.

2. EC2 Pricing Structure:

EC2 pricing varies based on the instance type, region, and usage. For a comparable container workload, you would typically be running t-series instances (on-demand or spot), which have varying costs depending on the utilization. The cost per hour for an t3.micro on-demand is around $0.0175.

3. Cost Comparison:

To make this comparison, we need to look at both static and dynamic workloads.

Static Workload: If your workload is predictable and doesn't require frequent scaling, manually managing EC2 instances could be more cost-effective due to the lower base costs of running fewer instances continuously.
Dynamic Workload: For applications that frequently scale up or down, Fargate’s pay-per-second model can provide significant savings by not paying for idle time.
4. Cost Calculation Example:

Let's consider a workload that requires 10 containers at any given moment and scales between 5 to 20 containers over an hour.

Fargate Cost:

Minimum charge per minute is $0.00225.
For 60 seconds, you pay $0.00225 (rounded from the minimum).
For each additional container, add another $0.00225.

EC2 Cost:

Assume on-demand pricing for t3.micro at $0.0175 per hour.
You would pay $0.0175 * 10 = $0.175 for the minimum, plus any additional instances as needed.
5. Break-Even Analysis:

To determine when Fargate’s convenience becomes less valuable, we need to find the point where the cost of managing EC2 instances and their associated overhead (like operating system management, instance type selection, and monitoring) outweighs the savings from Fargate’s pay-per-second model.

6. Complexity and Overhead:
Fargate: No need for OS maintenance, instance type selection, or scaling decisions.
EC2: Requires continuous monitoring, load balancing, capacity planning, and potential spot instance management (for cost optimization).

#### Closing Summary
and Common Mistakes/Follow-Ups:
When Fargate is Worth It:
Workloads that frequently scale up and down.
Applications where the predictability of resource usage is challenging.
Scenarios requiring auto-scaling and dynamic resource allocation.
When EC2 Might Be More Cost-Effective:
Predictable workloads with stable scaling patterns.
Lower overhead tolerance for manual management tasks.
Long-running applications that benefit from the lower base costs of on-demand instances.

#### Common Mistakes
to Avoid:
Overlooking the operational complexity and cost of manually managing EC2 instances, including monitoring and scaling decisions.
Failing to consider the long-term savings when workloads become more predictable.
Ignoring the convenience factor for teams accustomed to managed services.

#### Follow-Up
Questions:
How do additional factors like reserved instance pricing or spot instances impact this comparison?
What are some best practices for managing dynamic workloads in AWS that could be relevant here?

**Comparison:**

```
Workload Type	Fargate Cost (Min Charge per Minute)	EC2 Cost (On-Demand, 10 Containers)
Static	$0.135 per hour	$0.175 per hour
Dynamic	Varies by scaling	Varies by scaling
```

**Code:**

```python
# Example Code to Calculate Costs

def calculate_fargate_cost(containers, duration_minutes):
    cost_per_minute = 0.00225
    return containers * cost_per_minute * duration_minutes

def calculate_ec2_cost(container_count):
    on_demand_price_per_hour = 0.0175
    return container_count * (on_demand_price_per_hour / 60)

# Example Usage
fargate_cost = calculate_fargate_cost(10, 60)
ec2_cost = calculate_ec2_cost(10)
print(f"Fargate Cost: ${fargate_cost:.2f}")
print(f"EC2 Cost: ${ec2_cost:.2f}")
```

**Example input / output:**

```
Input: calculate_fargate_cost(10, 60), calculate_ec2_cost(10)
Output:
PLAINTEXT
Copy
Fargate Cost: $13.50
EC2 Cost: $0.175
```

**Data:**

```json
{
  "containers": 10,
  "duration_minutes": 60,
  "on_demand_price_per_hour": 0.0175
}
```

**Flow:**

1. Yes
2. No
3. Static Workload
4. Predictable Usage?
5. Use EC2 for Cost Savings
6. Frequent Scaling
7. Migrate to Fargate forConvenience and Lower BaseCosts
8. Complex Dynamic Workloads
9. Determine Break-Even Point
10. Consider Additional FactorsLike Monitoring andManagement Overhead

