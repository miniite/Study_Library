---
type: "practice"
difficulty: "Basic"
---

# What is a managed disk in Azure and what are the disk types (Standard HDD/SSD, Premium SSD)?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Subscriptions and Resource Hierarchy](<../../../../TOOLS/Azure/Azure_Platform/Subscriptions_and_Resource_Hierarchy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Azure, managed disks simplify the process of managing virtual machine (VM) storage by abstracting away many low-level details, allowing you to focus on your application needs rather than disk management intricacies. There are three main types of managed disks: Standard HDD, Standard SSD, and Premium SSD. Each type is optimized for different workloads based on performance, durability, and cost.

To begin, let's first understand the basic concept of a managed disk in Azure. A managed disk provides a highly available and durable storage option that can be attached to multiple VMs, making it ideal for scenarios where you need flexibility and reliability.

**Comparison:**

```
Disk Type	Performance Level	Description
Standard HDD	Moderate	Optimized for cost with moderate performance and durability.
Standard SSD	Higher	Offers better IOPS and throughput than Standard HDD, suitable for general-purpose workloads.
Premium SSD	Highest	Provides the highest level of performance with extremely low latency and high durability, ideal for SQL databases or other high-performance applications.
```

**Code:**

```python
# This is an illustrative example; actual code won't be needed as the concept explanation suffices.
```

**Example input / output:**

```text
Input:
- Creating a new managed disk in Azure

Output:
- Choosing between Standard HDD, Standard SSD, or Premium SSD based on workload requirements and cost considerations.
```

**Data:**

```json
{
  "diskTypes": [
    {
      "name": "Standard HDD",
      "description": "Optimized for cost with moderate performance and durability.",
      "performanceLevel": "Moderate"
    },
    {
      "name": "Standard SSD",
      "description": "Offers better IOPS and throughput than Standard HDD, making it suitable for general-purpose workloads.",
      "performanceLevel": "Higher"
    },
    {
      "name": "Premium SSD",
      "description": "Provides the highest level of performance with extremely low latency and high durability, ideal for SQL databases or other high-performance applications.",
      "performanceLevel": "Highest"
    }
  ]
}
```

**Flow:**

1. Managed Disk in Azure
2. Standard HDD
3. Standard SSD
4. Premium SSD

