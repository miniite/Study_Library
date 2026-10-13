---
type: "practice"
difficulty: "Deep"
---

# Your YARN cluster shows containers stuck in ACCEPTED state and jobs never start. Walk me through how you would debug queue capacity, scheduler config, and resource requests.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ YARN Troubleshooting](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/YARN_Troubleshooting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When debugging a Hadoop YARN cluster where containers are stuck in the "ACCEPTED" state and jobs never start, we need to systematically investigate several key areas: queue capacity, scheduler configuration, and resource requests. This involves a deep understanding of how YARN manages resources, the interaction between queues, and job scheduling mechanics.

Firstly, it's crucial to understand that when containers are stuck in ACCEPTED state, it usually indicates that there aren't enough resources available or that the cluster has reached its capacity limits for certain queues. Let's break down each component:

Queue Capacity: We need to check if the queue has sufficient capacity.
Scheduler Configuration: The scheduler can influence how and where containers are allocated.
Resource Requests: Jobs often specify resource requirements, which can affect whether they get scheduled.

Let’s start by triaging the issue step-by-step:

#### Triage
1. Queue Capacity

Queues in YARN control the allocation of resources among different applications or job types. If a queue is configured to have limited capacity, it may not be able to handle the number of incoming jobs.

#### Assumptions

Each container has specific resource requirements (CPU, memory).
Containers are being accepted but not launched due to lack of available slots.
2. Scheduler Configuration

The YARN scheduler determines how resources are allocated among different queues and applications. There are multiple schedulers like FIFO, CapacityScheduler, and FairScheduler, each with its own rules for resource allocation.

#### Assumptions

The cluster uses one of these schedulers.
There may be configuration parameters that could limit resource allocation or cause delays in job submission.
3. Resource Requests

Jobs submit resource requests based on their needs. If the requested resources exceed available capacity, jobs will remain in ACCEPTED state indefinitely until more resources become available.

#### First-Principles Solution

Check Queue Capacity:

Ensure that the queue does not have a hard capacity limit set.
Verify the number of active and pending containers against the maximum allowed for the queue.
Check if any other job in the same or higher-priority queue is consuming all available resources.

Review Scheduler Configuration:

Inspect the scheduler’s configuration files (e.g., capacity-scheduler.xml for CapacityScheduler).
Look for settings like yarn.scheduler.capacity.root.QueueA.maximum-capacity, which can restrict how much of the total capacity a queue can consume.
Ensure that the scheduler is configured to handle the expected workload and adjust parameters as needed.

Analyze Resource Requests:

Use YARN’s ResourceManager web UI or CLI tools like yarn application -list to examine job details and resource requests.
Verify that jobs are not requesting more resources than available in the cluster.

#### Cons
ider optimizing job configurations for better resource utilization, such as adjusting map/reduce tasks’ memory allocations.

#### Common Mistakes
and Follow-Ups
Common Mistake: Overlooking the queue capacity settings. Queue limits can be set to very low values by mistake.

#### Follow-Up
Questions:
How do you ensure that job requests are not overly greedy, causing resource starvation?
What steps would you take if dynamic scaling is enabled in your cluster?
Can you explain how YARN handles preemption of containers in the event of overallocation?

By systematically addressing these areas and leveraging YARN’s built-in monitoring tools, we can effectively debug and resolve issues related to container acceptance and job scheduling.

**Comparison:**

```
Queue	Maximum Capacity	Current Usage
root.default	100%	65%
root.mapreduce	50%	32%
root.spark	40%	7%
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Check Queue Capacity
6. Queue has capacity limit?
7. Verify active and pendingcontainers
8. Check for resource starvationin other queues
9. Review SchedulerConfiguration
10. Scheduler config correct?
11. Analyze Resource Requests
12. Tweak scheduler parameters
13. Monitor cluster performance
14. Dynamically scale resources ifneeded

