---
type: "practice"
difficulty: "Deep"
---

# How do you architect multi-tenant isolation in a shared data platform where a noisy tenant's query can starve others? Compare compute isolation, quotas, and separate accounts as strategies for a SaaS at scale.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant Data Platform Isolation](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Multi-tenant_Data_Platform_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of ensuring multi-tenant isolation in a shared data platform, particularly when a noisy tenant's query might starve others, we need to design an architecture that balances resource allocation, compute efficiency, and security. The strategies we will compare are compute isolation, quotas, and using separate accounts. Each has its own trade-offs and implications for scalability and cost management.

#### Triage
Understanding the Problem

The core issue is managing resources in a shared environment where multiple tenants coexist. If one tenant's queries consume disproportionately more resources than their allocated share, it can lead to performance degradation or even downtime for other tenants. We need to ensure that each tenant gets their fair share of compute resources while maintaining the overall system’s stability and efficiency.

#### Assumptions
Shared Data Platform: A cloud-native platform using services like Amazon Redshift, Google BigQuery, or a custom-built solution.
Noisy Tenant: A tenant whose queries are resource-intensive and could potentially monopolize the shared resources.
Multi-tenancy: Multiple tenants share the same infrastructure but require isolation to prevent each other from negatively impacting their workloads.

#### First-Principles Solution
Compute Isolation, Quotas, and Separate Accounts
1. Compute Isolation

#### Solution
Overview: Compute isolation involves dedicating specific resources or instances to individual tenants, thereby ensuring that even if a noisy tenant's query runs, it cannot consume all available resources. This can be achieved through containerization (e.g., Docker), virtual machines (VMs), or serverless functions.

Step-by-Step Solution:

Resource Allocation: Allocate specific compute instances or containers to each tenant.
Dedicated Resources: Ensure that the noisy tenant’s queries are run in their isolated environment, preventing them from accessing shared resources.
Monitoring and Logging: Implement detailed monitoring and logging to detect any resource hogging by a tenant.
2. Quotas

#### Solution
Overview: Quotas involve setting limits on the maximum usage of resources that each tenant can consume. This approach ensures that no single tenant can overwhelm the system, but it might not completely prevent noisy tenants from causing issues if they exceed their quota.

Step-by-Step Solution:

Define Quotas: Set per-tenant quotas for CPU, memory, disk I/O, and network bandwidth.
Enforcement Mechanisms: Implement mechanisms to enforce these quotas in real-time.
Grace Periods: Allow a grace period before enforcing hard limits, so that noisy tenants can be alerted and corrected.
3. Separate Accounts

#### Solution
Overview: Using separate accounts for each tenant ensures complete isolation but also adds complexity in terms of management and cost control. This approach is more secure but might not scale as efficiently as other methods.

Step-by-Step Solution:

Tenant-Specific Infrastructure: Create a dedicated account for each tenant, with their own resources.
Security Measures: Implement robust security measures to prevent cross-account attacks.
Cost Management: Monitor and manage costs associated with separate accounts, which can become expensive if not managed properly.

#### Closing Summary

Compute isolation provides the best balance between resource management and operational simplicity, while quotas offer a more flexible approach but require careful enforcement mechanisms. Separate accounts ensure complete isolation but come at a higher cost and complexity. The choice of strategy depends on the specific requirements of the platform, including tenant needs, resource constraints, and budget considerations.

#### Common Mistakes/Follow-Ups
Ignoring Monitoring: Failing to implement robust monitoring can lead to unnoticed resource hogs.
Overlooking Grace Periods: Not allowing grace periods for noisy tenants can result in unnecessary penalties or downtime.
Underestimating Cost Implications: Using separate accounts without proper cost management can lead to high operational costs.

#### Triage
Recap

The key is to implement a combination of strategies tailored to the platform’s needs. Compute isolation ensures immediate resource allocation, while quotas provide long-term stability and alerts. Separate accounts are useful for extremely sensitive or critical workloads but should be used judiciously due to their cost implications.

**Comparison:**

```
Strategy	Advantages	Disadvantages
Compute Isolation	Ensures immediate resource allocation.	Requires more operational overhead.
Quotas	Flexible and adaptable to varying workloads.	Can lead to underutilization of resources.
Separate Accounts	Complete isolation, security measures.	High cost, complex management.
```

**Code:**

```python
# Example of setting up compute isolation using Docker
import docker

client = docker.from_env()
container = client.containers.run("ubuntu", detach=True)
```

**Example input / output:**

```text
Input:
- Tenant 1 runs a resource-intensive query that consumes 95% of the CPU.

Output:
- The system detects the high CPU usage and sends alerts to Tenant 1.
- Compute isolation is applied, ensuring that Tenant 2’s resources are not affected.
```

**Data:**

```yaml
tenants:
  - tenant_id: "001"
    resources:
      cpu_limit: 2000
      memory_limit: 8GB
  - tenant_id: "002"
    resources:
      cpu_limit: 3000
      memory_limit: 6GB
```

**Flow:**

1. Yes
2. No
3. Exceeds
4. Within Limits
5. Tenant Query Starts
6. Is Tenant Noisy?
7. Apply Compute Isolation
8. Check Quotas
9. Enforce Quotas
10. Monitor Resources
11. Notify Tenant and ApplyIsolation
12. Grace Period - Alert Tenant

