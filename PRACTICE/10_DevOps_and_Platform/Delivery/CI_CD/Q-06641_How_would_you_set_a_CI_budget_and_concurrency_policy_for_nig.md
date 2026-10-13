---
type: "practice"
difficulty: "Intermediate"
---

# How would you set a CI budget and concurrency policy for nightly model retrains so a burst of retrains across many services doesn't starve regular deploy pipelines?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ ML Model Delivery Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/ML_Model_Delivery_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this CI/CD budgeting and concurrency policy question, we need to ensure that nightly model retrains are managed effectively without impacting regular deployment pipelines. The key is to balance resource allocation between these two critical processes.

#### Triaging the Question

The core of the question revolves around setting a budget for resources (like CPU and memory) and configuring a concurrency policy to manage the load from nightly model retrains, ensuring that they don’t overwhelm the system, thereby potentially starving regular deploy pipelines. The interviewer is probing your understanding of resource management in CI/CD pipelines, prioritization strategies, and how to handle burst loads.

#### Assumptions
Resource Availability: Assume a fixed set of resources available for both nightly model retrains and regular deployments.
Impact Analysis: Regular deployments have a known and consistent demand for resources.
Burst Load Characteristics: Nightly model retrains occur in bursts, potentially requiring more computational power than usual.

#### First-Principles Solution
Resource Budgeting:
Identify the typical resource requirements of both nightly model retrains and regular deployments.
Determine a baseline budget that covers the average demand for each process.
Concurrency Policy:
Implement a concurrency limit to control how many retrain tasks can run simultaneously.
Prioritize critical services, ensuring they get sufficient resources during their deployment windows.
Dynamic Scaling:
Utilize dynamic scaling to adjust resource allocation based on real-time demand.
Monitoring and Alerting:
Set up monitoring for resource usage and alerting mechanisms to notify when thresholds are breached.
Detailed Steps

Baseline Resource Allocation:

Assess the typical resource requirements for both nightly model retrains (e.g., 50 CPU units, 2GB memory) and regular deployments (e.g., 30 CPU units, 1GB memory).
Allocate a baseline budget that covers these demands.

Concurrency Limitation:

Set a maximum number of concurrent retrain jobs allowed at any time.
For example, limit to 5 simultaneous retrain tasks during the night.

Service Prioritization:

Identify critical services and ensure they always get priority access to resources.
Use service tagging or labels in your CI/CD tooling (e.g., is_critical: true).

Dynamic Scaling Mechanism:

Implement a dynamic scaling solution that adjusts the number of available CPU units and memory based on real-time demand.
For instance, if retrain jobs are running, scale up resources; otherwise, scale down to save costs.

Monitoring and Alerts:

Monitor resource usage in real time using tools like Prometheus or Grafana.
Set up alert rules to notify the team when resource thresholds are breached (e.g., 80% CPU utilization).

#### Closing Summary

By setting a budget for resources, implementing a concurrency policy that limits simultaneous retrain tasks, prioritizing critical services, and utilizing dynamic scaling, we can manage the load from nightly model retrains without starving regular deployment pipelines. This approach ensures efficient resource utilization while maintaining stability.

#### Common Mistakes/Follow-Ups
Ignoring Real-Time Demand: Failing to consider real-time demand for resources.
Overly Aggressive Concurrency Limits: Setting limits too low, which could starve regular deployments.
Lack of Monitoring and Alerts: Not having robust monitoring in place can lead to unnoticed resource issues.

Would you like to explore any specific aspects further, such as dynamic scaling strategies or alerting mechanisms?

**Code:**

```python
# Example Python script for dynamic resource scaling
import time

def scale_up(resources):
    print(f"Scaling up resources by {resources} units")
    # Simulate resource scaling logic here
    return True

def scale_down(resources):
    print(f"Scaling down resources by {resources} units")
    # Simulate resource scaling logic here
    return True

# Example function to handle retrain jobs
def manage_retrains(num_retries, max_concurrent=5):
    for i in range(num_retries):
        if i % 2 == 0:
            scale_up(10)  # Increase resources during even retries
        else:
            scale_down(5)  # Decrease resources during odd retries

manage_retrains(10)
```

**Data:**

```yaml
# Example YAML for service labels
services:
  - name: critical_service
    is_critical: true
  - name: non_critical_service
    is_critical: false
```

