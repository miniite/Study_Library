---
type: "practice"
difficulty: "Intermediate"
---

# Your etcd cluster backing a Kubernetes control plane is throwing 'apply request took too long' warnings and API latency is climbing. Walk me through how you'd triage whether it's disk fsync latency defragmentation need or raft leader instability.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Partition Lag and Hot Keys](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When troubleshooting a Kubernetes etcd cluster, we need to determine if high API latency and 'apply request took too long' warnings are due to disk fsync latency, defragmentation needs, or raft leader instability. By following a methodical triage process, we can isolate the root cause.

Triaging High API Latency in an Etcd Cluster
Understanding the Question

The etcd cluster backing your Kubernetes control plane is experiencing issues such as 'apply request took too long' warnings and increasing API latency. The goal is to identify whether the problem lies with disk performance, defragmentation needs, or raft leader instability.

#### Assumptions
We are dealing with a production etcd cluster in use by a Kubernetes control plane.
The cluster is deployed on nodes that have access to monitoring tools (like Prometheus, Grafana) and logging facilities.
Step-by-Step Triage

Gather Monitoring Data

Disk Performance Metrics: Check iops (input/output operations per second), io wait, and disk latency using tools like iostat, iotop, or Prometheus.
BASH
iostat -d 30
Raft Leader Election Metrics: Monitor raft leader election events, leader changes, and overall cluster health using etcd's internal metrics exposed via Prometheus.

Initial Analysis

Disk Fsync Latency:
Analyze io wait times to determine if there is a significant delay in writing data to the disk.
Check for large spikes or sustained high values, indicating potential issues with fsync operations.
Defragmentation Needs:
Examine the file system’s fragmentation level using tools like hdparm on Linux.
Monitor for frequent I/O operations that could indicate file system fragmentation.
Raft Leader Instability:
Look at the frequency and duration of raft leader changes. Frequent leader changes can impact API latency due to increased network overhead.

Detailed Investigation

Disk Fsync Latency:
Use iotop or similar tools to identify processes causing high disk I/O.
Check for any background operations that might be consuming resources, such as defragmentation tasks.
Defragmentation Needs:
Run a system-level check on file system fragmentation. For example, use badblocks and fsck commands.
BASH
sudo badblocks -v /dev/sda1 > /tmp/badblocks.txt
sudo fsck /dev/sda1 < /tmp/badblocks.txt
Raft Leader Instability:
Use etcd’s built-in health checks and leader election metrics to identify if there are frequent changes in the raft leadership.
Check for network issues or node failures that might be causing leader instability.

Mitigation Strategies

Disk Fsync Latency:
Optimize fsync settings or use persistent volumes with better performance characteristics (e.g., NVMe SSDs).
Defragmentation Needs:
Schedule defragmentation tasks during off-peak hours to avoid impacting cluster performance.
Raft Leader Instability:
Increase the raft heartbeat interval or add more etcd nodes for redundancy and improved stability.

#### Closing Summary

By systematically gathering monitoring data, analyzing key metrics, and investigating potential root causes, we can determine whether high API latency in an etcd cluster is due to disk performance issues, file system fragmentation, or raft leader instability. This approach ensures that the most effective mitigation strategies are employed to maintain a healthy and performant Kubernetes control plane.

#### Common Mistakes/Follow-Ups
Ignoring Disk Metrics: Failing to monitor disk I/O metrics can miss critical information about performance bottlenecks.
Overlooking Raft Health: Not paying attention to raft leader elections can lead to underestimating network issues or node failures.
Improper Defragmentation Scheduling: Performing defragmentation during peak hours can degrade cluster performance.

**Comparison:**

| Metric               | Thresholds                       | Action Taken                                          |
|----------------------|----------------------------------|------------------------------------------------------|
| Disk I/O Wait        | > 5%                             | Investigate fsync latency, consider SSDs             |
| Disk Latency         | > 10ms                           | Optimize disk performance or schedule defragmentation|
| Raft Leader Changes  | > 2 per minute                   | Adjust raft heartbeat intervals, add more nodes      |

**Code:**

```bash
# Example of iostat command output
iostat -d 30

# Example of badblocks command
sudo badblocks -v /dev/sda1 > /tmp/badblocks.txt
```

**Example input / output:**

```text
Input:
- etcd cluster is experiencing 'apply request took too long' warnings.
- API latency has increased.

Output:
- Disk fsync latency: Check iostat, io wait, disk latency metrics.
- Defragmentation needs: Run badblocks and fsck commands.
- Raft leader instability: Monitor raft leader changes and health checks.
```

**Data:**

```json
{
  "disk_metrics": {
    "io_wait": "5%",
    "disk_latency": "2ms"
  },
  "raft_metrics": {
    "leader_changes_per_second": 0.3,
    "leader_election_events": "12 per minute"
  }
}
```

**Flow:**

1. Start
2. Gather Metrics
3. Disk I/O Wait
4. Raft Leader Changes
5. Defragmentation Needs
6. Investigate fsync latency,consider SSDs
7. Adjust raft heartbeat intervals,add more nodes
8. Run badblocks and fsckcommands
9. Optimize disk performance orschedule defragmentation
10. Monitor network health,ensure redundancy
11. Schedule defragmentationduring off-peak hours

