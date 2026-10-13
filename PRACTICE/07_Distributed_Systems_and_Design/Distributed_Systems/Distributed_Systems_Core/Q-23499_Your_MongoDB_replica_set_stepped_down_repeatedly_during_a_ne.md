---
type: "practice"
difficulty: "Intermediate"
---

# Your MongoDB replica set stepped down repeatedly during a network hiccup, flapping the primary. Explain the election timeout heartbeat and priority settings behind this and how you'd stabilize it.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Partition Lag and Hot Keys](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

The issue with your MongoDB replica set is likely due to repeated elections caused by the network hiccup, leading to rapid flapping between primaries. To address this, we need to understand and adjust the election timeout and heartbeat settings.

#### Triage

The question probes an understanding of MongoDB's replica set mechanics, specifically how electionTimeoutSeconds, heartbeatInterval, and members' priority affect stability during network issues. We will assume a standard setup with three nodes in the replica set.

#### First Principles Solution

Election Timeout (electionTimeoutSeconds):

This is the maximum time a node waits to hear from other nodes before considering them down.
If this period is too short, it can lead to frequent elections and potentially rapid flapping between primaries.

Heartbeat Interval:

MongoDB uses heartbeats to detect failures. The default interval is 100ms (0.1 seconds).
Frequent heartbeats ensure quick detection of node failure but may not be sufficient for very short network disruptions.

Member Priority:

Higher priority members have a greater chance of becoming the primary.
Setting an appropriate priority can prevent rapid elections by giving certain nodes more weight.
Step-by-Step Solution

Adjust electionTimeoutSeconds:

Increase this value to 20 seconds (the default is 10 seconds). This gives nodes more time to communicate and reduces the likelihood of false elections.
JSON
{
  "_id": "my-replica-set",
  "members": [
    { "_id" : 0, "host" : "node-0.example.com", "priority" : 1 },
    { "_id" : 1, "host" : "node-1.example.com", "priority" : 1 },
    { "_id" : 2, "host" : "node-2.example.com", "priority" : 1 }
  ],
  "settings": {
    "electionTimeoutMillis": 20000
  }
}

Optimize heartbeatInterval:

While the default is good, you might want to reduce it to 50ms for very short network disruptions.
JSON
"settings": {
  "heartbeatIntervalMillis": 50,
  "electionTimeoutMillis": 20000
}

Set Member Priorities:

Assign higher priority (e.g., 2) to critical nodes or those with more resources.
JSON
{
  "_id": "my-replica-set",
  "members": [
    { "_id" : 0, "host" : "node-0.example.com", "priority" : 1 },
    { "_id" : 1, "host" : "node-1.example.com", "priority" : 2 }, // Higher priority
    { "_id" : 2, "host" : "node-2.example.com", "priority" : 1 }
  ]
}

#### Closing Summary
and Common Mistakes

By increasing the electionTimeoutSeconds, reducing the heartbeatIntervalMillis, and setting appropriate member priorities, you can stabilize your replica set during network hiccups. Common mistakes include leaving the default settings unaltered, which may not be sufficient for environments with unreliable networks.

#### Follow-Ups
Network Quality: Ensure consistent network quality by using tools like ping or network monitoring.
Disk I/O Latency: Monitor disk I/O latency; high delays can affect elections.
Node Configuration: Check node configurations to ensure they are optimized and not under heavy load.

