---
type: "practice"
difficulty: "Deep"
---

# How do you choose between the Capacity Scheduler and the Fair Scheduler for a multi-tenant Hadoop cluster shared by analytics and ETL teams?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Hadoop and Batch Processing](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/Hadoop_and_Batch_Processing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a multi-tenant Hadoop cluster where both analytics (interactive queries, ad-hoc jobs) and ETL (batch processing with high throughput requirements) teams share resources, choosing between the Capacity Scheduler and Fair Scheduler is crucial to ensure optimal resource utilization and performance for all users. The Capacity Scheduler allows administrators to specify shares of capacity among different queues, while the Fair Scheduler aims to provide fair treatment by ensuring that each job gets a fair amount of resources based on its priority.

The key decision factors include:

Job Characteristics: Analytical queries often require interactive response times and may not be sensitive to resource sharing, whereas ETL jobs prioritize throughput.
Resource Requirements: ETL tasks usually have predictable and high resource demands, while analytical queries can vary significantly in size and duration.
User Prioritization: Different teams might have different SLAs or priorities that need to be respected.

To make the best choice:

Analyze Job Profiles: Identify which type of job (analytical vs. ETL) is more prevalent and what their typical resource requirements are.
Queue Configuration: Design queue structures in both schedulers, considering how they handle priority and share allocation.
Performance Metrics: Monitor cluster performance and user satisfaction with the chosen scheduler to validate your decision.

Let's break this down step by step from first principles:

#### Triaging the Question

The question is asking us to choose between two scheduling policies: Capacity Scheduler (CS) and Fair Scheduler (FS). The key aspects to consider are:

Resource Allocation: How resources are distributed among different queues or users.
User Experience: Ensuring that both analytics and ETL teams get adequate performance.
Flexibility and Predictability: CS offers more control over resource allocation, while FS is designed for fairness.

#### Assumptions
Both teams have varying job types: Analytics (interactive, short-lived queries) vs. ETL (batch processing, long-running tasks).
The cluster has multiple queues or tenants.
Performance metrics are available to measure the effectiveness of the chosen scheduler.

#### Solution
from First Principles
Capacity Scheduler
Key Features:
Administrators can define queues and allocate specific percentages of resources to each queue.
Queues can have a minimum share, ensuring that critical tasks always get some resources.
Offers more flexibility in managing resource allocation based on business needs.

#### Example
Configuration:

Admin Queue
Analytics
ETL
Scenario: If the ETL team has high throughput requirements and the analytics team prefers interactive response times, CS can be configured to prioritize ETL by allocating more resources to its queue.
Fair Scheduler
Key Features:
Ensures that each job gets a fair share of resources.
Uses a combination of priority and resource usage history to schedule jobs fairly among users.
Designed for environments where all users have the same SLA and need guaranteed service levels.

#### Example
Configuration:

Admin Queue
Analytics
ETL
Scenario: If both teams require fair treatment and can tolerate some variance in job execution time, FS might be a better choice as it ensures that no single user monopolizes resources.
Choosing Between Them

To decide between CS and FS:

Queue Structure:
Define queues for each team to ensure isolation of workloads.
Priority and SLA:
Set appropriate priorities based on the criticality of jobs in each queue.
Resource Allocation:
Use CS if you need more control over resource allocation, or FS if fairness is paramount.

#### Closing Summary

In summary, the choice between Capacity Scheduler and Fair Scheduler depends heavily on the job profiles and performance requirements of both analytics and ETL teams. Capacity Scheduler offers more flexibility and control for administrators, making it suitable when specific priorities and SLAs need to be enforced. On the other hand, Fair Scheduler ensures that all jobs get a fair share of resources, which is ideal in environments where fairness and balanced resource usage are key.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking job characteristics: Both teams might have different needs, so it’s essential to tailor the scheduler configuration accordingly.

#### Follow-Up
Monitoring cluster performance regularly can help identify if one scheduler is better suited for the environment over time. Metrics like average task execution times and queue waits can guide future adjustments.

**Comparison:**

```
Queue	Capacity	Minimum User Limit %
analytics	40%	25
etl	60%	100
```

**Code:**

```xml
<!-- Example Capacity Scheduler XML Configuration -->
<configuration>
  <property>
    <name>yarn.scheduler.capacity.root.QueueA.maximum-capacity</name>
    <value>60</value>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.QueueB.minimum-user-limit-percent</name>
    <value>50</value>
  </property>
</configuration>
```

**Example input / output:**

```text
Input:
- User analytics-job submits a query.
- User etl-job submits a large ETL task.

Output:
- Analytics job gets allocated resources from the 'analytics' queue.
- ETL job gets high priority in the 'etl' queue, ensuring it runs faster than other jobs.
```

**Data:**

```json
{
  "queues": [
    {
      "name": "analytics",
      "capacity": 40,
      "minimum-user-limit百分比": 25
    },
    {
      "name": "etl",
      "capacity": 60,
      "minimum-user-limit百分比": 100
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Job Submission
4. Queue Assignment
5. Is Job ETL?
6. High Priority in 'etl' Queue
7. Low Priority in 'analytics'Queue

