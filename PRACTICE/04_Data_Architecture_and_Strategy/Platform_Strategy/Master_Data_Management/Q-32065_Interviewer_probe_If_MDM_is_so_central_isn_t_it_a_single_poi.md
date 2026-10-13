---
type: "practice"
difficulty: "Expert"
---

# Interviewer probe: 'If MDM is so central, isn't it a single point of failure?' Walk through your HA/DR design and RTO/RPO targets for the hub and its event streams.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ MDM for Batch and Streaming](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/MDM_for_Batch_and_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Master Data Management (MDM) is indeed crucial but can be designed with high availability (HA) and disaster recovery (DR) strategies to mitigate the risk of it becoming a single point of failure. In this scenario, we are discussing the hub architecture of an MDM system and its event streams, focusing on achieving our Recovery Time Objective (RTO) and Recovery Point Objective (RPO).

The primary challenge is ensuring that if the central hub or any of its connected nodes fail, the system can recover quickly and efficiently. We will outline a robust design to achieve this by leveraging multiple data centers, redundant systems, and automated recovery mechanisms.

#### Triaging the Question

The interviewer is probing into how we architect an MDM solution that ensures minimal downtime (RTO) and zero data loss (RPO), even when faced with critical failures. They are interested in understanding our approach to high availability, disaster recovery, and the trade-offs involved in these strategies.

#### Assumptions
Multiple Data Centers: We assume a distributed MDM system with multiple geographically dispersed data centers.
Event Streams: The MDM system uses event streams for real-time updates and changes in master data.
RTO Target: 5 minutes: Our target is to recover the hub and its connected nodes within 5 minutes of failure.
RPO Target: 1 minute: We aim to have zero or minimal data loss, with a maximum acceptable RPO of 1 minute.

#### First-Principles Solution

To achieve our HA/DR design targets, we will implement the following strategies:

1. Multi-Data Center Architecture:

We will use a multi-data center approach where each data center contains a complete copy of the MDM hub and its event streams. This ensures that if one data center fails, another can take over seamlessly.

Data Center 1
Hub Node
Event Stream
Database
Data Center 2
Hub Node
Event Stream
Database
2. Hub and Spoke Topology:

Each data center will have a hub node responsible for managing the master data and orchestrating updates through event streams.

FLOWCHART · MERMAID
graph LR;
    A[Data Center 1] --> H1[Hub Node]-> C1[Event Stream]-> J1[Spoke Nodes];
    B[Data Center 2] --> E1[Hub Node]-> F1[Event Stream]-> K1[Spoke Nodes];
3. Real-Time Synchronization:

We will use real-time synchronization mechanisms like Apache Kafka or AWS Kinesis to ensure that the hub and spoked nodes are constantly updated. This ensures that data is consistent across all nodes.

A
Kafka Topic
J1
B
Kafka Topic
K1
4. Automated Failover Mechanism:

In case of a failure, the failover mechanism will automatically switch the primary hub to another node in a different data center. This can be achieved using Kubernetes or container orchestration tools.

A
Failover Controller
B1
B
Failover Controller
E1
5. Backup and Restore Strategy:

We will implement regular backups of the database and event streams to an offsite location using tools like AWS S3 or Azure Blob Storage. In case of a catastrophic failure, these backups can be used to restore the system.

FLOWCHART · MERMAID
graph LR;
    A --> M1[Backup Service] -> N1[Offsite Backup];
    B --> O1[Backup Service] -> P1[Offsite Backup];
6. Monitoring and Alerting:

Continuous monitoring of the hub, spoked nodes, and event streams will be in place to detect anomalies early. Alerts will be configured for critical failures and system performance issues.

A
Monitoring Service
Alerts
B
Monitoring Service
Alerts

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, our MDM solution will be architected with multiple data centers and redundant systems to ensure high availability. Real-time synchronization mechanisms will maintain consistency across all nodes, while automated failover and backup strategies will minimize RTO and RPO. Continuous monitoring and alerting will help in quick detection and resolution of issues.

#### Common mistakes
include:

Overlooking real-time synchronization mechanisms.
Not having a robust backup strategy for catastrophic failures.
Failing to implement effective monitoring and alerting systems.

#### Follow-ups
might include discussing specific technologies, costs associated with the architecture, and how this solution scales as the system grows.

**Comparison:**

```
Data Center	Hub Nodes	Event Streams	Spoke Nodes
DC1	H1	C1	J1
DC2	E1	F1	K1
```

**Code:**

```python
# Example of a simple failover controller in Kubernetes
def detect_failure(node):
    if is_down(node):
        switch_to_backup_node()

def is_down(node):
    # Check node status using health checks
    return True

def switch_to_backup_node():
    # Use Kubernetes or similar to switch the primary hub
    pass
```

**Example input / output:**

```text
Input: Hub Node A failure detected.
Output: Failover to Backup Node B initiated.
RTO = 5 minutes, RPO = 1 minute achieved.
```

**Data:**

```yaml
# Example of a simple backup configuration for databases and event streams
backup:
  schedule: "0 * * * *"
  destination: s3://backup-bucket
```

**Flow:**

1. Hub Node Failure
2. Alert Sent
3. Failover Controller Triggered
4. Switch to Backup Node
5. RTO = 5 minutes

