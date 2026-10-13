---
type: "practice"
difficulty: "Expert"
---

# Architect capacity for a once-a-year event (say a mega-sale) that is 100x normal load for 10 minutes — how do you avoid paying for 100x capacity year-round while still guaranteeing the peak survives?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Real-time Price Feeds](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Real-time_Price_Feeds.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect capacity for a once-a-year mega-sale event that spikes to 100 times normal load for 10 minutes, our goal is to design a system capable of handling this massive load without overprovisioning during the rest of the year. The key here is to balance between availability and cost by leveraging cloud resources efficiently.

#### Triage
What's Being Probed

The question is asking about how to manage peak capacity while minimizing unnecessary costs, effectively probing your understanding of scalable systems, cost optimization strategies, and real-world trade-offs in architecture design. Key assumptions include the event’s short duration (10 minutes) and the availability requirements during this period.

#### First-Principles Solution

We'll start by breaking down the problem into its core components:

Identify Peak Load: Estimate how much normal traffic translates to 100x load.
Determine On-Demand Scalability Needs: Calculate if on-demand resources can handle this spike without constant overprovisioning.
Design a Hybrid Architecture: Blend pre-provisioned capacity with on-demand scaling.
Implement Cost Management and Monitoring: Ensure cost stays within acceptable limits while maintaining required performance.
Step-by-Step Solution

Estimate Peak Load:

Assume the normal load is 50,000 requests per second (RPS). Therefore, during the mega-sale, we expect to see 5 million RPS.

Determine On-Demand Scalability Needs:

For a duration of 10 minutes, the total request volume would be (5 \text{ million} \times 600 = 3 \times 10^9) requests.
Cloud providers often charge for compute and storage on an hourly basis. To avoid paying for constant 100x capacity, we need to design a solution that can scale up during the event.

Design a Hybrid Architecture:

Pre-Provisioned Capacity: Start with a baseline of 50,000 RPS handled by your existing infrastructure.
Auto-Scaling Groups (ASGs): Use ASGs to dynamically adjust the number of instances based on real-time load metrics. Configure ASG rules to scale out and in during the event.
Load Balancers: Employ a high-capacity load balancer that can handle increased traffic without failing, distributing requests efficiently among backend servers.

Implement Cost Management and Monitoring:

Use cost optimization tools provided by cloud providers (e.g., AWS Budgets) to monitor spending during the event.
Set up alerts for unexpected spikes in spend and configure automatic scaling policies that trigger cost-saving actions if thresholds are exceeded.

#### Closing Summary

By designing a hybrid architecture that combines pre-provisioned capacity with on-demand scaling, we can ensure peak performance during the mega-sale while keeping costs under control. Key elements include precise load estimation, effective use of auto-scaling groups, and robust cost management strategies.

#### Common Mistakes
and Follow-Ups
Overprovisioning: Always tempted to provision too much capacity at a higher cost.
Lack of Monitoring: Failing to set up proper monitoring can lead to unexpected costs or service disruptions.
Insufficient Planning: Not accounting for the nuances of auto-scaling, such as cold start times and warm-up periods.

#### Follow-ups
might include:

How would you implement advanced caching strategies?
What about database scalability during such a peak load?
Discuss failover mechanisms in case of ASG failures.

**Comparison:**

```
Component	Description
Pre-Provisioned Capacity	Baseline infrastructure to handle normal traffic.
Auto-Scaling Groups (ASGs)	Dynamically adjust the number of instances based on real-time load metrics, ensuring peak performance during the mega-sale.
Load Balancers	Distribute requests efficiently among backend servers and high-capacity load balancer.
Cost Optimization Tools	Monitor spending and set up alerts for unexpected spikes to prevent overprovisioning.
```

**Code:**

```
MERMAID
Copy
graph LR
    A[Load Estimation] --> B(50,000 RPS)
    B --> C[Auto-Scaling Groups]
    C --> D[High-Capacity Load Balancer]
    D --> E[Backend Servers]
    F[Caching Strategy] --> D
    G[Database Scaling] --> D
```

**Example input / output:**

| Input | Output |
|-------|--------|
| 50,000 RPS (normal) | 5 million RPS (mega-sale peak) |
| Total requests during mega-sale | 3 x 10^9 |

**Data:**

```json
{
  "normal_load": 50000,
  "mega_sale_load": 50000 * 100,
  "total_requests_mega_sale": 50000 * 600
}
```

**Flow:**

1. No
2. Yes
3. Normal Load (50,000 RPS)
4. Is Mega-Sale?
5. Continue with Pre-ProvisionedCapacity
6. Load Balancer DistributesRequests
7. Backend Servers ScaleUp/Down Automatically
8. Ensure High PerformanceDuring Mega-Sale

