---
type: "practice"
difficulty: "Deep"
---

# What's the difference between Flink's session, application, and per-job deployment modes, and which fits a multi-tenant observability platform?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Zero-loss Job Upgrades](<../../../../TOOLS/Apache_Flink/Flink_Core/Zero-loss_Job_Upgrades.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a multi-tenant observability platform, deploying Apache Flink can involve several different modes to manage resources and ensure performance isolation among tenants. The key differences between Flink's session, application, and per-job deployment modes are critical to understand when designing such a system.

Firstly, let’s break down the question: the interviewer is probing your understanding of how these deployment modes affect resource management, task isolation, and overall platform scalability in a multi-tenant environment. We will assume that we're dealing with a system where multiple tenants (each potentially with their own distinct requirements) need to share resources efficiently without interfering with each other.

Session Deployment Mode

In session mode, Flink runs as a standalone process, similar to how you might run it on a local machine or in a development environment. This mode is the simplest and most straightforward for small-scale deployments where resource contention between different jobs isn’t an issue. However, in a multi-tenant setting, this mode doesn't provide any built-in isolation mechanisms.

Application Deployment Mode

The application deployment mode allows you to manage multiple jobs within a single Flink cluster but under distinct namespaces or application IDs. Each application can have its own resource allocations and configurations, which is crucial for maintaining separate environments per tenant. This mode supports better isolation compared to session mode because it ensures that different applications do not interfere with each other by default.

Per-Job Deployment Mode

Per-job deployment mode is the most advanced in terms of resource management and isolation. In this model, Flink runs each job as a separate instance, ensuring that resources are isolated at the finest granularity. This means that even within an application namespace, different jobs can be deployed independently without sharing any state or resources.

Triaging the Modes for Multi-Tenant Observability
Session Mode: Not suitable due to lack of isolation and resource management.
Application Mode: Better suited but still requires careful configuration to ensure tenants don’t inadvertently affect each other.
Per-Job Mode: The most robust approach as it ensures that even within a single tenant, different jobs or streams are completely isolated.

#### First-Principles Solution

To implement an effective multi-tenant observability platform using Flink, we would choose the per-job deployment mode. This setup allows us to:

Isolate Resources: Each job can be allocated specific resources (CPU, memory) without affecting others.
Ensure Data Privacy: Since jobs run in isolation, data from one tenant cannot leak into another’s environment.
Scalability and Performance: We can scale each job independently based on its load requirements.

#### Closing Summary

In summary, per-job deployment mode is the best fit for a multi-tenant observability platform as it provides the highest level of resource isolation and data privacy. Application mode offers better isolation than session mode but still requires careful management to prevent interferences between tenants. Session mode is not recommended due to its lack of isolation.

#### Common Mistakes
Misconfiguring Resource Allocation: Not properly setting up resource limits can lead to performance bottlenecks or resource exhaustion.
Ignoring Isolation Issues: Failing to ensure that jobs run independently can result in data leakage between tenants.
Overlooking Scalability Considerations: Assuming a one-size-fits-all approach without considering varying load requirements of different tenants.

**Comparison:**

```
Deployment Mode	Isolation Level	Resource Allocation	Data Privacy
Session	Low	Shared	Poor
Application	Medium	Per-Application	Moderate
Per-Job	High	Per-Job	Strong
```

**Code:**

```java
// Example Flink Job Configuration for Per-Job Mode
FlinkJob job = new FlinkJob();
job.setDeploymentMode(DeploymentMode.PER_JOB);
job.allocateResources(1024, 512); // Allocate 1GB of memory and 5 cores
```

**Example input / output:**

```text
Input:
- Tenant A stream data (A1)
- Tenant B stream data (B1)

Output:
- Tenant A processed data (A1_processed)
- Tenant B processed data (B1_processed)
```

**Data:**

```json
{
  "tenantA": {
    "streamId": "A1",
    "resources": {"memory": 1024, "cpus": 5}
  },
  "tenantB": {
    "streamId": "B1",
    "resources": {"memory": 768, "cpus": 3}
  }
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Session Mode
6. Does Isolation Matter?
7. Low Resource Allocation
8. Application Mode
9. Per-tenant ResourceManagement?
10. Resource Contention
11. Per-Job Mode
12. High Isolation and DataPrivacy

