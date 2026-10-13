---
type: "practice"
difficulty: "Deep"
---

# How does NameNode high availability work with an active and a standby NameNode?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Small Files in HDFS](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/Small_Files_in_HDFS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

High Availability (HA) in Hadoop, specifically with respect to the NameNode, is crucial for maintaining cluster reliability without manual intervention. The NameNode is the central component responsible for managing metadata and coordinating data access requests across DataNodes. In an HA setup, we have two NameNodes—active and standby—which take turns being active based on their states and health.

To understand how this works, let's break it down from first principles.

**Code:**

```
MERMAID
Copy
stateDiagram-v2
    state "Active" as A
        --> B
    state "Standby" as B
        --> A
    A --> "Heartbeat from Active" : "heartbeat"
    B --> "Health check passes" : "health check"
```

**Example input / output:**

```text
Input (Initial State):
Active NameNode: Heartbeat time - 2023-10-01T14:00Z
Standby NameNode: Health check passed at 2023-10-01T15:00Z

Output (Transition to Standby):
Active NameNode transitions to Standby due to no heartbeat.
Standby NameNode becomes active and takes over as the new Active NameNode.
```

**Data:**

```json
{
  "nameNodeStates": {
    "active": {
      "state": "Active",
      "lastHeartbeatTime": "2023-10-01T14:00Z"
    },
    "standby": {
      "state": "Standby",
      "healthCheckPassed": true,
      "lastHealthCheckTime": "2023-10-01T15:00Z"
    }
  }
}
```

