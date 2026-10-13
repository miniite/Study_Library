---
type: "practice"
difficulty: "Deep"
---

# How would you capacity-plan a serverless/autoscaling database (Aurora Serverless v2, DynamoDB on-demand) where you're planning cost envelopes rather than fixed instances?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Capacity Planning and Throughput](<../../../../01_Databases_and_SQL/Database_Operations/Capacity_Planning/Capacity_Planning_and_Throughput.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Capacity planning for a serverless or autoscaling database like Amazon Aurora Serverless v2 or DynamoDB On-Demand requires a deep understanding of both the workload characteristics and the cost implications. The key is to ensure that you can meet performance requirements while keeping costs within budget. In this scenario, we're focusing on cost envelopes rather than fixed instance sizes.

Firstly, let's clarify what the interviewer is probing: they want to understand how you approach capacity planning in a cost-conscious manner for serverless databases, considering both performance and financial constraints. Key assumptions include:

The database workload will have varying demand.
You need to balance between meeting service level agreements (SLAs) and staying within budget.
Cost envelopes are defined based on predefined thresholds or SLA targets.

To solve this, we'll break it down step by step:

Define the Workload: Understand the transactional characteristics of your application, including read/write throughput, latency requirements, and peak loads. Use historical data to predict future usage patterns.
Set Cost Targets: Define your cost envelopes based on different SLA targets (e.g., 95th percentile response time). For example, you might have a low-cost envelope for normal operations and a high-cost envelope if performance degrades significantly.
Monitor and Analyze Performance Metrics: Continuously monitor key metrics such as read/write throughput, latency, and error rates. Use these to identify when the system is approaching its cost envelopes.
Adjust Auto-Scaling Policies: Based on real-time data, adjust the auto-scaling policies to ensure performance while keeping costs under control. This might involve setting up alerts for cost thresholds and manually scaling resources if necessary.
Optimize Queries and Indexes: Ensure that queries are optimized and indexes are well-designed to minimize read/write operations and reduce latency.
Utilize Cost Management Tools: Leverage AWS cost management tools like Cost Explorer, Budgets, and Reserved Instances to manage costs effectively.

By following these steps, you can ensure that your serverless database is both performant and cost-effective.

**Comparison:**

```
Metrics	Thresholds	Actions
Latency	>500 ms	Scale Up
Throughput	<100 RU/s	Scale Down
```

**Code:**

```python
# Example Python code snippet for monitoring and adjusting auto-scaling policies

def monitor_and_adjust(db_client, current_metrics):
    # Get the latest performance metrics from AWS CloudWatch
    metrics = db_client.get_performance_metrics()

    if metrics['latency'] > high_threshold:
        # Scale up resources to meet higher demand
        response = db_client.scale_up()
        print(f"Scalability action taken: {response}")
    elif metrics['readThroughput'] < low_threshold:
        # Scale down resources if demand is low
        response = db_client.scale_down()
        print(f"Scalability action taken: {response}")
```

**Example input / output:**

```text
Input:
- Current metrics from CloudWatch for a serverless database instance

Output:
- Action to scale up or down based on predefined thresholds
- Response message indicating the action taken, e.g., "Scaling up resources due to high latency"
```

**Data:**

```json
{
  "low_threshold": 100,
  "high_threshold": 500
}
```

**Flow:**

1. Latency > 500ms
2. ReadThroughput < 100RU/s
3. Monitor Metrics
4. Monitor CloudWatch
5. Check Thresholds
6. Scale Up
7. Scale Down

