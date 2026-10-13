---
type: "practice"
difficulty: "Expert"
---

# What is tiered storage in Kafka (KIP-405), and how does it change how you size local disks in 2025?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Claim Check Pattern](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Claim_Check_Pattern.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Tiered storage in Apache Kafka, as defined by KIP-405, is a strategy that allows Kafka to store messages across different types of storage layers (e.g., SSDs and HDDs), optimizing both cost and performance based on the age of data. By 2025, this concept will significantly influence how we size local disks in Kafka clusters, shifting our focus towards leveraging cost-effective long-term storage while maintaining fast access to recent messages.

To understand tiered storage, let’s first revisit KIP-405: it proposes adding a new log.retention.bytes parameter and a log.flush.interval.messages parameter that can be used to configure retention policies for different segments of the log. This allows Kafka to intelligently manage data placement based on its age and access frequency.

#### Triage

The question is probing how we can effectively use tiered storage in 2025, specifically focusing on disk sizing. The key assumptions are:

Storage technologies will have evolved by 2025.
We need a strategy to balance cost and performance.
Kafka needs to support efficient data management over extended periods.

#### First Principles Solution

Understanding the Problem: By 2025, traditional local disk sizes for Kafka brokers may be smaller due to advancements in storage technologies like NVMe SSDs and larger, slower HDDs or even cloud storage options. Tiered storage allows us to store older data on cheaper, higher-capacity storage while keeping newer, more frequently accessed data on faster storage.

Designing the Strategy:

RetentionPolicy Configuration: Use log.retention.bytes and log.flush.interval.messages to define how long Kafka retains data in each storage tier. For example, recent messages can be retained for a shorter period on SSDs, while older messages are moved to HDDs or cloud storage.
Automation with Broker Management Tools: Integrate Kafka’s management tools to automatically move data between tiers based on the configured policies.
Performance Optimization: Ensure that read and write operations prioritize accessing faster storage layers. Use caching mechanisms like in-memory caches (e.g., Redis) for highly frequent reads.

#### Implementation
Steps:

Define the storage tiers: Identify which types of data should reside where based on access patterns.
Configure Kafka settings: Update server.properties to enable tiered storage and configure retention policies.
Test and Monitor: Set up monitoring and logging to ensure that data is correctly being moved between storage tiers without performance degradation.

#### Closing Summary

By 2025, the use of tiered storage in Kafka will revolutionize how we manage local disks, shifting towards more cost-effective solutions while maintaining high performance for critical data. This approach requires careful planning and configuration but can lead to significant savings and improved operational efficiency.

#### Common Mistakes/Follow-Ups
Over-reliance on Local Storage: Some may underestimate the need for cloud storage or external storage solutions, leading to suboptimal cost management.
Inadequate Performance Testing: Without thorough testing, there's a risk of degraded performance when moving data between different tiers.
Ignoring Data Consistency: Ensuring that data consistency is maintained across multiple storage layers can be challenging and requires careful design.

**Comparison:**

```
Tier	Storage Type	Retention Period	Performance
SSD	NVMe	<1 day	High
HDD	Sata	1-7 days	Medium
Cloud	AWS/S3	>7 days	Low
```

**Code:**

```python
# Example configuration snippet in server.properties for tiered storage
log.flush.interval.messages=1000
log.retention.bytes=50G
```

**Example input / output:**

```text
Input:
- New message: Retained on SSD for 5 minutes.
- Old message: Moved to HDD after 24 hours.

Output:
- Fast access to recent data.
- Cost-effective long-term storage solution.
```

**Data:**

```json
{
  "storage.tiers": [
    {
      "name": "SSD",
      "type": "SSD",
      "retention.bytes": "20GB"
    },
    {
      "name": "HDD",
      "type": "HDD",
      "retention.bytes": "1TB"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Message Produced
6. Is Message Recent?
7. Store on SSD
8. Is Disk Full?
9. Migrate to HDD
10. Migrate to Cloud
11. Monitor Performance
12. Maintain Consistency

