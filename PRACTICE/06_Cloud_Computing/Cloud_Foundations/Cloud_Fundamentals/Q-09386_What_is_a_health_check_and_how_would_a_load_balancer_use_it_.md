---
type: "practice"
difficulty: "Basic"
---

# What is a health check and how would a load balancer use it to stop sending traffic to a broken model replica?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Health Checks and Load Balancers](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Health_Checks_and_Load_Balancers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Health checks are crucial mechanisms in cloud environments, especially when deploying models across multiple replicas in a load-balanced setup. They ensure that only healthy and functioning instances receive incoming traffic while broken or faulty ones are isolated to avoid service degradation. In the context of ML engineers, understanding how a load balancer uses health checks is vital for maintaining system reliability.

To understand this concept, we first need to clarify what a health check entails. At its core, a health check involves periodic verification of an instance's availability and readiness to handle requests. For an ML model, these checks might involve sending small, predefined data samples through the model and evaluating the output against expected results or certain thresholds.

Now, let's dive into how a load balancer would use these health checks:

Understanding the Health Check Mechanism: A load balancer periodically sends test requests to each model replica. These tests can be as simple as sending dummy input data through an API endpoint and checking if the response is within acceptable ranges or follows expected behavior.

Setting Up Health Checks:

Define a threshold for health, such as an error rate below 1%.
Configure the load balancer to send test requests at regular intervals (e.g., every few seconds).
If a replica fails more than the defined threshold, it is marked as unhealthy.

Actioning Unhealthy Instances: When a model replica is identified as unhealthy:

The load balancer stops sending traffic to that instance.
It may mark this state in its health check logs and possibly trigger alerts.
Optionally, the system can attempt to restart or redeploy the replica if automated recovery mechanisms are in place.

Rebalancing Traffic: Once a replica is deemed healthy again, it is reintroduced back into the load-balanced pool of instances.

In summary, health checks ensure that only reliable and functioning model replicas receive traffic, thereby maintaining high service quality and uptime.

#### Common mistakes
to avoid:

Not setting up robust health check protocols can lead to continuous misrouting of traffic.
Overlooking the importance of timely failover mechanisms in case an instance remains unhealthy for too long.

