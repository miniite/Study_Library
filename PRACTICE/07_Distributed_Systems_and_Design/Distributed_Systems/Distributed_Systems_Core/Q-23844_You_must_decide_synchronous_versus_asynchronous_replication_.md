---
type: "practice"
difficulty: "Deep"
---

# You must decide synchronous versus asynchronous replication of a fine-tuning checkpoint store across regions for DR. For a job you can't afford to fully restart, argue the durability-versus-latency trade-off and what RPO you'd accept.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sync vs Async Replication](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Sync_vs_Async_Replication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide between synchronous versus asynchronous replication of a fine-tuning checkpoint store across regions for disaster recovery (DR), we need to carefully balance the critical requirements of durability against latency, considering factors like acceptable Recovery Point Objective (RPO) and business impact.

We are tasked with ensuring that a job cannot be fully restarted if it fails, implying high importance on data availability. The decision hinges on understanding the trade-off between immediate availability and potential loss of recent updates due to replication delays.

Synchronous replication ensures that all replicas are updated before the write operation completes, providing stronger consistency guarantees but potentially introducing latency. Asynchronous replication allows writes to complete immediately, with delayed updates, offering lower latency at the cost of weaker consistency.

Let’s break down the trade-offs and assumptions:

#### Assumptions

We have multiple data centers or regions available for replication.
The job cannot be restarted fully if it fails, indicating that recent fine-tuning checkpoints are critical.
RPO must meet business requirements; for example, an RPO of 5 minutes might be acceptable in some applications.

Durability vs. Latency Trade-Off:

Synchronous Replication: This ensures immediate durability across all replicas, minimizing the risk of data loss. However, it introduces additional latency due to waiting for acknowledgments from multiple regions.
Asynchronous Replication: This offers lower latency as writes can complete faster but may result in a delay before updates are fully replicated.

Given these factors, we would accept an RPO that aligns with the criticality of the job and available infrastructure. For instance, if the job is mission-critical and must not lose recent checkpoints, synchronous replication might be necessary to ensure data integrity at the cost of increased latency.

#### Example Scenario
If our system can tolerate up to 5-minute RPO for checkpoint updates due to the nature of the fine-tuning process, we could opt for asynchronous replication. This allows quicker write operations but ensures that within a maximum delay (e.g., 5 minutes), the data will eventually be consistent across regions.

In summary, choosing synchronous over asynchronous replication depends on our ability to accept a higher RPO and the criticality of ensuring no loss of recent checkpoints. Synchronous replication provides stronger consistency at the expense of increased latency, while asynchronous replication offers lower latency but with potential delays in data availability.

**Comparison:**

```
Replication Method	Latency (ms)	Durability	RPO (minutes)
Synchronous	High	High	0
Asynchronous	Low	Medium	Up to N
```

**Code:**

```python
# Example code for managing synchronous vs. asynchronous replication
class CheckpointReplicator:
    def __init__(self, sync_threshold=5):
        self.sync_threshold = sync_threshold

    def replicate_checkpoint(self, checkpoint_data, target_regions):
        if len(target_regions) > 1 and self.sync_threshold != 0:
            # Asynchronous replication
            for region in target_regions:
                region.replicate(checkpoint_data)
            return "Replicated asynchronously"
        else:
            # Synchronous replication
            for region in target_regions:
                region.replicate(checkpoint_data, wait=True)
            return f"Replicated synchronously with RPO {self.sync_threshold} minutes"
```

**Example input / output:**

```text
Input:
Replicator = CheckpointReplicator(sync_threshold=5)
CheckpointData = {"model_version": "v2.3", "timestamp": "2023-10-01T14:30"}
TargetRegions = ["us-west-2", "eu-central-1", "ap-southeast-1"]

Output:
"Replicated asynchronously"
```

**Data:**

```json
{
  "checkpoint_store": {
    "regions": ["us-west-2", "eu-central-1", "ap-southeast-1"],
    "replication_method": "asynchronous",
    "rpo_minutes": 5
  }
}
```

**Flow:**

1. Yes
2. No
3. Initial Write
4. Is Replication Synchronous?
5. Synchronize with all Regions
6. Asynchronous Write, Wait forAcknowledgment
7. Data Consistent Across AllRegions
8. Write Completed, DataAvailable in One Region Only
9. Check RPO Compliance

