---
type: "practice"
difficulty: "Expert"
---

# You must set RPO/RTO tiers for the observability estate across metrics, logs, traces, and alerting. Explain why alerting is effectively tier-0 (losing it means flying blind) while cold trace archives can tolerate more loss, and how you'd map replication topology to each tier and stop everyone claiming tier-0.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Sync vs Async Replication](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Sync_vs_Async_Replication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In AIOps architecture, setting RPO (Recovery Point Objective) and RTO (Recovery Time Objective) tiers is critical to ensure observability of metrics, logs, traces, and alerting. Alerting stands out as the most crucial component, effectively acting as tier-0 because its loss leads to operational blindness. In contrast, cold trace archives can tolerate more data loss without significant impact on operations. By understanding these priorities, we can design a robust replication topology that ensures high availability for critical components while allowing for cost-effective storage of less vital data.

#### Triaging the Question

The question is asking us to define and prioritize RPO/RTO tiers in an observability estate across various data types: metrics, logs, traces, and alerting. The focus on tier-0 highlights that alerting must be prioritized above all else due to its critical nature. We need to explain why this is the case and how different replication topologies can be applied based on these priorities.

#### First Principles Solution
Alerting as Tier-0: Operational Blindness

Alerting systems are the first line of defense in monitoring operational health. Losing alerting means that any issues or anomalies go unnoticed, leading to severe consequences such as system downtime, data loss, or security breaches. Therefore, ensuring no loss of alerting is paramount; this makes it a tier-0 requirement.

Cold Trace Archives: Tolerating More Loss

Cold trace archives, on the other hand, are historical records that can be accessed for post-mortem analysis and compliance purposes. Their loss would result in reduced visibility but not immediate operational issues, making them more flexible regarding RPO/RTO requirements.

Replication Topology Mapping

To map these priorities to a replication topology, we must design the system to ensure high availability for critical components while allowing cost-effective storage of less vital data.

Critical Components (Tier-0):

Alerting: Ensure zero data loss by using synchronous replication.
Metrics and Logs: Use asynchronous or near-real-time replication with minimal latency.

Less Critical Components:

Cold Trace Archives: Utilize asynchronous replication to tolerate longer recovery times, reducing storage costs.
Design Considerations
Synchronous Replication for Tier-0 (Alerting): Synchronous replication guarantees that data is written to both the primary and secondary nodes simultaneously, ensuring zero loss. This approach ensures high availability but can introduce latency.
Asynchronous Replication for Tier-1 (Metrics/Logs): Asynchronous replication writes data to the secondary node at a later time, allowing for lower latency compared to synchronous methods while still providing strong reliability guarantees.
Cold Trace Archives: Use a combination of asynchronous replication and periodic archiving to balance availability with cost. This approach allows for high availability during operational hours but can tolerate longer recovery times.

#### Closing Summary

By prioritizing alerting as tier-0, we ensure the system remains operationally viable under any circumstances. Cold trace archives can be designed with more relaxed RPO/RTO requirements, allowing for cost-effective storage and retrieval of historical data. This tiered approach ensures that critical operational functions are always available while optimizing resource allocation.

#### Common Mistakes
Claiming Tier-0 as an Overhead: Sometimes organizations might treat alerting as a low-priority component, which can lead to significant risks in operations.
Neglecting Cold Trace Archives: Failing to properly archiving historical data can result in operational issues when troubleshooting or auditing.
Ignoring Latency Impacts: Not accounting for the latency introduced by asynchronous replication can impact real-time decision-making processes.
Triaging Follow-Ups
Failure Scenarios: Discuss potential failure modes and how they affect each tier.
Cost vs. Availability Trade-offs: Evaluate different cost models and their impact on RPO/RTO requirements.
Scalability Considerations: Assess the scalability of various replication strategies to ensure they can handle large volumes of data.

**Comparison:**

```
Tier	Data Type	RPO (Seconds)	RTO (Minutes)
Tier-0	Alerting	0	<1
Tier-1	Metrics, Logs	30	5
Tier-2	Cold Trace Archives	3600	1440
```

**Code:**

```python
# Example Code for Synchronous Replication
def replicate_data_synchronously(primary_node, secondary_node):
    # Ensure both nodes are updated simultaneously
    primary_node.update_data(data)
    secondary_node.update_data(data)

# Asynchronous Replication
def replicate_data_asynchronously(secondary_node, data):
    # Update the secondary node with a delay
    time.sleep(random.randint(1, 5))  # Simulating random delays
    secondary_node.update_data(data)
```

**Example input / output:**

```text
Input (Example):
- Alerting data received at 09:00 AM
- Metrics and logs data received every minute

Output (Expected):
- Alerting data replicated synchronously by 09:00 AM
- Metrics and logs data replicated asynchronously with a maximum delay of 1 second
- Cold trace archives updated once per hour, with the latest data available within an hour
```

**Data:**

```json
{
  "alerting": {
    "rpo": "0 seconds",
    "rto": "< 1 minute"
  },
  "metrics_logs": {
    "rpo": "30 seconds",
    "rto": "5 minutes"
  },
  "cold_trace_archives": {
    "rpo": "1 hour",
    "rto": "24 hours"
  }
}
```

**Flow:**

1. Alerting
2. Synchronous Replication
3. Primary Node
4. Secondary Node
5. Metrics, Logs
6. Asynchronous Replication
7. Primary Node
8. Secondary Node
9. Cold Trace Archives
10. Periodic Archiving
11. Archival Storage

