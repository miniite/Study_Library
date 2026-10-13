---
type: "practice"
difficulty: "Deep"
---

# Describe how you'd build the cost model for a migration from a self-managed database to a managed cloud DB, including the hidden costs (egress, IOPS, backups, cross-AZ, license) that make the naive comparison misleading. How do you present the honest TCO?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ TCO of Self-managed Platforms](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/TCO_of_Self-managed_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Building a cost model for migrating from self-managed databases to managed cloud databases involves identifying and quantifying all relevant costs, including hidden expenses like egress fees, IOPS requirements, backup strategies, cross-AZ replication, and licensing terms. The key is to present an honest Total Cost of Ownership (TCO) that accounts for both direct and indirect expenses over a reasonable period.

#### Triaging the Question

The question asks us to design a cost model for migrating from self-managed databases to managed cloud databases while highlighting hidden costs that often lead to misinformed comparisons. The interviewer is probing your ability to understand comprehensive cost considerations, present accurate TCO calculations, and make strategic recommendations based on detailed analysis.

#### Assumptions
Current Environment: Self-managed database on-premises or in a legacy cloud.
Target Environment: Managed cloud database service (e.g., AWS RDS, Google Cloud SQL).
Timeframe: One year for comparison and three years for more comprehensive TCO analysis.
Data Size: 1 TB of data to be migrated initially with expected growth.
Usage Patterns: Peak usage time is during business hours.
Step-by-Step Solution
1. Identify Direct Costs

Self-managed Database Costs:

Hardware costs for servers, storage, and network.
Maintenance and operational overhead (e.g., DBA salaries, security patches).
Power and cooling infrastructure.

Managed Cloud Database Costs:

Storage fees per GB per month.
CPU and memory usage charges.
Network egress costs.
2. Hidden Costs in Self-managed Databases
Egress Fees: If you are using a public network to transfer data between on-premises and the cloud, consider egress rates.
IOPS Requirements: Ensure your current workload can be supported by the IOPS limits of the target database service. Over-provisioning might increase costs.
Backup Strategy:
On-premises backup storage and network bandwidth.
Cloud-based backup and restore operations.
3. Hidden Costs in Managed Databases
Cross-AZ Replication: Ensuring high availability with cross-AZ replication can add costs, especially if it’s required.
License Costs: Some managed services have licensing fees for data volume or performance.
4. Cost Calculation

Self-managed Database:

PLAINTEXT
Total Cost = (Hardware + Maintenance) * Number of Servers + Network Egress Fees + Security and Patching

Managed Cloud Database:

PLAINTEXT
Total Cost = Storage Costs + CPU/Memory Usage Charges + Network Egress Costs + Backup and Restore Operations + Cross-AZ Replication Costs + Licensing Fees
5. Presenting the TCO
Yearly Comparison: Calculate both models for a one-year period to understand immediate impacts.
Long-term Analysis: Extend the analysis over three years to capture ongoing operational costs and potential growth.

#### Example
Calculation

Assume:

On-premises storage: $10,000 per server annually
Self-managed database maintenance: $5,000 per server annually
Egress fees for data transfer: $200 per GB
Cloud storage costs: $0.35 per GB per month
Cloud CPU and memory usage charges: $0.12 per hour
Network egress in cloud: $0.16 per GB
PLAINTEXT
On-premises:
Total Cost = (Hardware + Maintenance) * 4 Servers + Egress Fees * Data Size
           = ($10,000 + $5,000) * 4 + 200 * 1,000
           = $72,000

Managed Cloud:
Total Cost = Storage Costs * 36 months + CPU/Memory Usage Charges + Egress Fees + Cross-AZ Replication Costs + Licensing Fees
           = (0.35 * 1,000 * 36) + (0.12 * 8760 * 4) + 200 * 1,000 + Cross-AZ + Licensing
           = $15,120 + $4,464 + 200,000 + Cross-AZ + Licensing

#### Closing Summary

By carefully considering all direct and hidden costs, we can build a realistic cost model that accurately reflects the total cost of ownership for both self-managed and managed database solutions. This analysis helps in making informed decisions about migration strategies.

#### Common Mistakes/Follow-ups
Omitting Hidden Costs: Failing to account for egress fees, IOPS requirements, or backup strategies can lead to underestimated costs.
Ignoring Long-term Implications: Only comparing short-term costs might miss long-term savings from cloud services.
Overlooking Licensing Terms: Some managed databases have licensing fees that need to be factored in.
Triaging Follow-ups
How would you adjust the cost model if the self-managed database is running on a different hardware configuration?
What metrics would you use to justify moving back to self-managed databases after the initial migration?
Can you discuss any case studies where your organization faced similar challenges and how they were resolved?

**Comparison:**

| Cost Component | On-premises (Yearly) | Managed Cloud (Yearly) |
|----------------|---------------------|-----------------------|
| Storage        | $40,000             | $15,120               |
| Maintenance    | $20,000             | N/A                   |
| Egress Fees    | $800                | $4,464                |
| Cross-AZ       | N/A                 | N/A                   |
| Licensing      | N/A                 | $160                  |
| **Total**      | **$72,000**         | **$20,144**           |

**Code:**

```python
# Example Python code for cost calculation
def calculate_on_premise_cost(num_servers, storage_cost_per_server, maintenance_cost):
    return (storage_cost_per_server + maintenance_cost) * num_servers

def calculate_cloud_cost(storage_cost_per_gb_month, cpu_memory_usage_hours, egress_cost_per_gb):
    storage_cost = 36 * storage_cost_per_gb_month
    usage_cost = cpu_memory_usage_hours * 0.12
    return storage_cost + usage_cost + (egress_cost_per_gb * 1000)
```

**Example input / output:**

```text
On-premises Cost Calculation:
Input:
- Storage cost per server: $10,000
- Maintenance cost: $5,000
- Number of servers: 4

Output:
Total On-premises Cost = (10000 + 5000) * 4 + 200 * 1000
                        = 72,000

Managed Cloud Cost Calculation:
Input:
- Storage cost per GB month: $0.35
- CPU/Memory usage hours: 8760 * 4
- Egress cost per GB: $0.16

Output:
Total Cloud Cost = (36 * 0.35) + (8760 * 4 * 0.12) + (0.16 * 1000)
                 = 15,120 + 4,464 + 160
                 = 20,144
```

**Data:**

```json
{
  "onPremiseCost": {
    "storageCostPerServer": 10000,
    "maintenanceCost": 5000,
    "numServers": 4,
    "egressCostPerGB": 200
  },
  "cloudCost": {
    "storageCostPerGBMonth": 0.35,
    "cpuMemoryUsageHours": 8760 * 4, // 1 year of usage
    "egressCostPerGB": 0.16
  }
}
```

**Flow:**

1. Yes
2. No
3. Identify Direct Costs
4. Are there hidden costs?
5. Account for all hidden costs
6. Proceed with cost model
7. Present TCO Analysis
8. Finalize Cost Model
9. Presentation of Honest TCO

