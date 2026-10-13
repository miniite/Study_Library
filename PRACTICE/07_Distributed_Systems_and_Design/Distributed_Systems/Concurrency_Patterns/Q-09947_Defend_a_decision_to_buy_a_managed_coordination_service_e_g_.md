---
type: "practice"
difficulty: "Expert"
---

# Defend a decision to buy a managed coordination service (e.g., a hosted consensus/lock offering) versus running your own ZooKeeper/etcd fleet. Frame it as a build-vs-buy at org scale: TCO, operational blast radius, vendor lock-in, and the concurrency guarantees you can't compromise.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Coordination Choices](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Distributed_Coordination_Choices.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When deciding between buying a managed coordination service like AWS Managed Control Tower or running your own ZooKeeper/etcd fleet, we need to consider Total Cost of Ownership (TCO), operational blast radius, vendor lock-in, and the critical concurrency guarantees required for our system. For large-scale organizations, a managed solution often provides better TCO, reduced operational risk, and critical guarantees that may be harder or costlier to achieve on your own.

#### Triaging the Question

The core of this question is about evaluating whether it’s more advantageous to implement a distributed coordination service in-house versus leveraging a managed external service. The key factors include:

Total Cost of Ownership (TCO): This covers all costs associated with running and maintaining the system.
Operational Blast Radius: The impact on other systems if something goes wrong.
Vendor Lock-in: Dependence on third-party services and potential restrictions or costs for switching providers.
Concurrency Guarantees: The level of consistency and reliability required for your applications.

#### Assumptions
We are dealing with a large-scale organization that requires high availability, strong consistency, and low operational overhead.
The managed service offers similar functionalities to ZooKeeper/etcd but with additional support, monitoring, and redundancy built-in.
There is no significant difference in the base functionality between running our own solution and using a managed one.
Step-by-Step Solution
1. Total Cost of Ownership (TCO)

In-House Implementation:

Initial Setup: Requires hiring specialized engineers to set up, configure, and monitor ZooKeeper/etcd clusters.
Maintenance Costs: Ongoing costs for maintaining the infrastructure, including hardware upgrades, software patches, and security updates.
Operational Overhead: Continuous management of backups, failovers, and disaster recovery processes.

Managed Service:

Initial Setup: Minimal setup required as the service is managed by the vendor.
Maintenance Costs: Reduced maintenance costs due to the provider’s expertise in managing the infrastructure.
Scalability: Automatic scaling and redundancy provided by the service, reducing the need for manual interventions.
2. Operational Blast Radius

In-House Implementation:

Single Point of Failure: If a node fails, it can take down the entire cluster unless redundant configurations are implemented.
Manual Recovery: Manual steps required to recover from issues, increasing downtime and recovery time.
Limited Monitoring Tools: Limited visibility into system health and performance.

Managed Service:

Redundancy and Failover: Built-in redundancy and automatic failover mechanisms reduce the risk of complete service outages.
Automated Recovery: Automatic monitoring and recovery processes minimize downtime.
Enhanced Visibility: Better tools for monitoring and diagnosing issues, reducing troubleshooting time.
3. Vendor Lock-in

In-House Implementation:

No Lock-in Risk: You can always use other coordination services if needed in the future.
Customization: Ability to customize the setup according to specific requirements without vendor constraints.

Managed Service:

Vendor Lock-in Risk: Dependence on the provider’s service, which could become a constraint for future changes or migrations.
Integration Challenges: Potential challenges integrating with other services provided by the same vendor due to lock-in policies.
4. Concurrency Guarantees

In-House Implementation:

Complexity of Implementation: Ensuring strong consistency and atomic transactions can be complex and error-prone.
Resource Intensive: High resource requirements for maintaining high availability and performance in a distributed system.

Managed Service:

Guaranteed Consistency: Managed services typically offer strong consistency guarantees, reducing the risk of data corruption or inconsistencies.
Reliability: Built-in reliability features like automatic failover and retries ensure that your applications can handle failures gracefully.

#### Closing Summary

For large-scale organizations with stringent requirements for concurrency and operational efficiency, a managed coordination service is often the more favorable option. It offers better TCO, reduced operational risk, and critical guarantees that may be hard to achieve in-house. However, it's important to carefully evaluate the trade-offs, especially concerning vendor lock-in.

#### Common Mistakes
and Follow-ups
Overestimating In-House Capabilities: Underestimating the complexity and resource requirements of running a distributed system.
Ignoring Operational Costs: Failing to account for ongoing maintenance and operational overhead.
Vendor Lock-in Ignored: Assuming that in-house solutions will always be easier to change or migrate.

**Comparison:**

```
	Managed Service	In-House Service
Initial Setup Cost	$0	$5,000
Maintenance Cost	$1,000/year	$3,000/year
```

**Code:**

```python
# Example pseudo-code for comparing setup costs between managed and in-house services

class ManagedServiceCost:
    def initial_setup_cost(self):
        return 0  # No initial setup required

    def maintenance_cost(self):
        return 1000  # Assumed low ongoing cost due to managed service

class InHouseSetupCost:
    def initial_setup_cost(self):
        return 5000  # High initial cost for hiring and setting up infrastructure

    def maintenance_cost(self):
        return 3000  # Higher ongoing costs due to manual management
```

**Example input / output:**

```text
Managed Service:
- Initial Setup Cost: $0
- Maintenance Cost: $1,000/year

In-House Service:
- Initial Setup Cost: $5,000 (hiring and infrastructure)
- Maintenance Cost: $3,000/year
```

**Data:**

```json
{
  "managed_service": {
    "initial_setup_cost": 0,
    "maintenance_cost": 1000
  },
  "in_house_service": {
    "initial_setup_cost": 5000,
    "maintenance_cost": 3000
  }
}
```

**Flow:**

1. No
2. Yes
3. Managed Service
4. Initial Setup
5. Low Ongoing Costs
6. Maintenance Cost: $1k/year
7. In-House Service
8. High Initial Cost
9. Manual Management
10. Maintenance Cost: $3k/year

