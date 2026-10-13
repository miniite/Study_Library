---
type: "practice"
difficulty: "Deep"
---

# How do you design an org-wide 'compute placement' decision framework so that VM-vs-container-vs-serverless is chosen from workload shape and cost curve rather than each team's tribal preference?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ GPU Quota Management](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/GPU_Sharing/GPU_Quota_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Designing a compute placement decision framework involves understanding the workload characteristics, cost considerations, and technical constraints to ensure scalable, efficient, and cost-effective resource allocation across an organization. The key is to align these decisions with workload profiles—ranging from VMs for stateful workloads, containers for microservices, and serverless functions for event-driven architectures—and leverage a cost curve analysis.

The core of the solution involves building a robust data pipeline that collects performance metrics, monitoring tools, and financial dashboards. By analyzing this data, teams can make informed decisions based on workload patterns and cost optimization rather than adhering to tribal preferences.

#### Triage

The question asks us to design a framework for deciding between VMs, containers, and serverless functions based on the shape of the workload and its associated cost curve.
The interviewer is probing our ability to integrate technical expertise with business considerations, such as cost optimization and resource efficiency. They also want to ensure that the solution considers scalability, performance, and security.

#### Assumptions
We need a comprehensive understanding of various workloads, cost models, and organizational requirements.

#### Solution

Workload Characterization: Begin by characterizing each workload based on its nature (stateful vs. stateless), predictability, and resource utilization patterns. For instance:

VMs are suitable for applications that require persistent storage and complex configurations.
Containers are ideal for microservices or apps with varying demand patterns but no need for persistent storage.
Serverless functions are appropriate for event-driven workloads where the application’s scaling is handled automatically.

Cost Curve Analysis: Develop a cost model that factors in both operational costs (e.g., compute, storage, network) and non-operational costs (e.g., developer time, maintenance). This involves creating a cost curve for each type of deployment:

VMs: Typically higher upfront costs but lower per-hour costs.
Containers: Lower initial setup but higher per-container management overhead.
Serverless: Pay-as-you-go with low per-request costs but variable costs based on request volume.

Data Collection & Analysis: Implement a comprehensive data collection system that monitors key performance indicators (KPIs) such as:

CPU and memory utilization
Network I/O
Disk I/O
Application response times

Performance Metrics & Dashboards: Use metrics like request latency, error rates, and throughput to fine-tune the cost models and adjust deployments accordingly.

Decision Tree & Automation: Design a decision tree that uses historical data and real-time metrics to determine the most appropriate compute resource for each workload. Automate this process where possible to reduce human intervention and ensure consistency.

Governance & Compliance: Ensure that any chosen solution adheres to organizational policies, compliance requirements, and security standards.

Scalability & Resilience: Consider future growth and scaling needs when deploying resources. Ensure the framework can adapt as workloads change over time.

#### Closing Summary
By designing a compute placement decision framework grounded in workload characteristics, cost analysis, and real-time performance data, organizations can achieve more efficient resource utilization and cost savings. This approach not only optimizes resource allocation but also fosters a culture of best practices across different teams.

#### Common Mistakes/Follow-Ups

Failing to adequately characterize the workloads: Understanding the nature of each workload is crucial for making accurate decisions.
Overlooking non-operational costs: Often, hidden costs like developer time and maintenance can significantly impact overall costs.
Underestimating scalability and resilience requirements: Ensuring that the framework can adapt to changing conditions is essential for long-term success.

**Comparison:**

```
Compute Resource	Cost Model Factors	Scalability Considerations
VM	Fixed setup + hourly	Easier horizontal scaling
Container	Startup + hourly	Can handle dynamic scaling
Serverless	Per-request	Auto-scaling
```

**Code:**

```python
# Example of a basic cost model function
def calculate_cost(workload_type, hours_used):
    if workload_type == 'vm':
        return 0.1 * hours_used + 50  # $0.1 per hour + fixed setup cost
    elif workload_type == 'container':
        return 0.02 * hours_used + 5  # $0.02 per container-hour + startup cost
    elif workload_type == 'serverless':
        return 0.01 * requests * 0.0004  # $0.01 per request + variable cost

# Example usage
cost_vm = calculate_cost('vm', 50)
print(f"VM cost for 50 hours: {cost_vm}")
```

**Example input / output:**

```text
Input:
- Workload metrics collected over a day for each workload type

Output:
- Recommended compute placement decision based on cost and performance analysis
- Example output: "For the given workload, recommend using containers due to its lower per-hour cost while meeting required performance criteria."
```

**Data:**

```json
{
  "workload_metrics": [
    {
      "workload_type": "vm",
      "cpu_utilization": 80,
      "memory_usage": 7GB,
      "network_io": 10MB/s
    },
    {
      "workload_type": "container",
      "cpu_utilization": 50,
      "memory_usage": 3GB,
      "network_io": 5MB/s
    },
    {
      "workload_type": "serverless",
      "requests_per_minute": 1000,
      "response_time": 2ms
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Characterize Workload
6. Is Stateful?
7. Use VMs
8. Is Predictable?
9. Use Containers
10. Use Serverless Functions
11. Collect Performance Metrics
12. Analyze Cost Curve
13. Make Decision Based onAnalysis
14. Automate DeploymentAdjustments

