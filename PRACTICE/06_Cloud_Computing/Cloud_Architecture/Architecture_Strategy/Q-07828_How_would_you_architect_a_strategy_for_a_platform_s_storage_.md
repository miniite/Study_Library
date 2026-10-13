---
type: "practice"
difficulty: "Expert"
---

# How would you architect a strategy for a platform's storage tiering (hot/warm/cold/archive) across object, block, and file, optimizing for cost without hurting recovery SLAs?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Region Failure Resilience](<../../../../06_Cloud_Computing/Cloud_Architecture/Architecture_Strategy/Region_Failure_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a cost-optimized storage tiering strategy for a platform that supports hot, warm, cold, and archive data tiers using object, block, and file storage, we need to balance cost with recovery SLAs. This involves understanding the nature of the data, its access patterns, and the underlying cloud services available.

#### Triage

The question is probing your ability to design a scalable, efficient, and cost-effective storage tiering strategy that meets specific performance and recovery requirements. Key assumptions include:

We are working with cloud-based storage solutions.
The platform supports object, block, and file storage types.
Recovery SLAs must be strictly adhered to.
Cost optimization is the primary goal.

#### First Principles Solution

Identify Data Tiers:

Hot Tier: High-frequency access data (e.g., recent backups).
Warm Tier: Medium-frequency access data (e.g., old backups, infrequently accessed logs).
Cold Tier: Low-frequency access data (e.g., historical records, older backups).
Archive Tier: Very low-frequency or rarely accessed data (e.g., long-term archives).

Storage Types and Their Characteristics:

Object Storage: Ideal for cold and archive tiers due to its cost-effectiveness and scalability.
Block Storage: Suitable for warm and hot tiers as it offers better performance compared to object storage but is more expensive.
File Storage: Can be used across all tiers, but may not provide the best performance or cost.

Cost Optimization:

Use object storage for cold and archive tiers where access patterns are infrequent.
Employ block storage for hot and warm tiers to meet high performance requirements at a reasonable cost.
Balance file storage usage carefully, as it can be costly and may not offer the best performance.

Recovery SLAs:

Ensure that recovery times from cold and archive tiers do not exceed acceptable limits by optimizing data retrieval mechanisms.
Use advanced features like multi-region replication or caching to speed up access in case of emergency.

Automation and Orchestration:

Implement automated tiering policies based on access patterns to move data between tiers as needed.
Leverage cloud services for backup and recovery, ensuring data is replicated across regions if necessary.

#### Closing Summary

By strategically utilizing different storage types—object, block, and file—we can optimize costs while meeting strict recovery SLAs. The key is to align the right type of storage with appropriate access patterns and leverage automation to manage tiering efficiently.

#### Common Mistakes/Follow-Ups
Ignoring Performance Requirements: Failing to consider that hot tiers require high performance, which could lead to higher costs.
Over-Reliance on Object Storage: Not considering file or block storage for warm tiers where there might be a need for better performance.
Lack of Automation: Manual tiering can become cumbersome and may not scale well over time.

Would you like me to elaborate on any specific aspect, such as multi-region replication strategies or cost models?

**Comparison:**

```
Tier	Type	Cost per GB	Performance
Hot	Block	$0.20	High
Warm	Block	$0.15	Moderate
Cold	Object	$0.025	Low
Archive	Object	$0.01	Very low
```

**Code:**

```python
# Example Python script to simulate automated tiering policy
def move_data_to_tier(data, tier):
    if tier == 'hot':
        # Move data to block storage with high IOPS and low latency
        pass
    elif tier == 'warm':
        # Move data to block storage with moderate IOPS and acceptable latency
        pass
    elif tier == 'cold':
        # Move data to object storage for cost optimization
        pass
    elif tier == 'archive':
        # Move data to glacier or similar low-cost storage class
        pass

# Example usage
data = "recent_backup"
move_data_to_tier(data, 'hot')
```

**Example input / output:**

```text
Input:
- Data access pattern for a 'recent_backup' (high frequency)
- Recovery SLA requirement of 30 minutes

Output:
- Move 'recent_backup' to block storage tier 'hot'
- Ensure backup is replicated across multiple regions for disaster recovery within the required SLA
```

**Data:**

```json
{
  "storage_tiers": {
    "hot": {
      "type": "block",
      "cost_per_gb": 0.2,
      "performance": "high"
    },
    "warm": {
      "type": "block",
      "cost_per_gb": 0.15,
      "performance": "moderate"
    },
    "cold": {
      "type": "object",
      "cost_per_gb": 0.025,
      "performance": "low"
    },
    "archive": {
      "type": "object",
      "cost_per_gb": 0.01,
      "performance": "very low"
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Hot Tier
8. High Frequency?
9. Block Storage
10. Warm Tier
11. Medium Frequency?
12. Block Storage
13. Cold Tier
14. Low Frequency?
15. Object Storage
16. Archive Tier

