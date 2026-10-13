---
type: "practice"
difficulty: "Basic"
---

# What does the bootstrap.servers list actually do, and why is it fine to list only a subset of brokers?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Leader Election and Unclean Failover](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Leader_Election_and_Unclean_Failover.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The bootstrap.servers list in Kafka is crucial as it tells the client which broker nodes to connect to initially when establishing the connection to the Kafka cluster. While listing all brokers is ideal, it's perfectly fine and often more practical to specify only a subset of them because:

Reducing the number of initial connections can improve performance by minimizing network overhead.
It provides redundancy: if one of the listed brokers fails or becomes unresponsive, the client can attempt to connect to another one in the list.

Let's break down these points with more detail:

TL;DR

The bootstrap.servers list in Kafka is used to establish initial connection to any of the specified broker nodes. Listing only a subset ensures faster startup and better resilience against failures while maintaining robust connectivity.

#### Triaging the Question

The question probes your understanding of:

How clients connect to Kafka clusters.
The purpose and implications of specifying bootstrap.servers.

#### Trade-offs
between listing all brokers vs. a subset in terms of performance and reliability.
From First Principles
Understanding Kafka Clients

Kafka clients use a distributed architecture where producers send messages to topics, which are managed by brokers (nodes). Consumers read from these topics by connecting to the broker nodes. The bootstrap.servers list is used during client initialization as a starting point for establishing connections.

Role of bootstrap.servers
Initialization: Clients first contact one of the specified brokers listed in bootstrap.servers to gather information about the entire cluster.
Resilience and Load Balancing: By listing only a subset, clients can reduce initial connection overhead and still be resilient as they will try connecting to another broker if needed.
Why It's Fine to List Only a Subset

Reduced Initial Overhead:

Listing all brokers might cause more network traffic during startup.
For large clusters with many brokers, this could delay the client’s initialization time.

Redundancy and Failover:

If one of the listed brokers is down or unresponsive, the client can still try connecting to another broker in the list.
This ensures that even if some nodes are temporarily unavailable, the application continues to function without needing manual intervention.

Scalability:

As clusters grow, listing all brokers becomes cumbersome and might not be necessary for initial connection attempts.
Specifying a subset helps manage the number of connections during startup, reducing the load on both the client and the broker nodes.

#### Conclusion

In summary, while it is possible to list all available brokers in bootstrap.servers, specifying only a subset of them offers practical benefits such as reduced initial overhead and better resilience. This approach balances performance with robustness without compromising the overall functionality of the Kafka cluster.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the trade-offs between listing all or just a few brokers.

#### Follow-Up
Discussing scenarios where listing all brokers might be necessary, e.g., in small clusters or highly dynamic environments.

#### Closing Summary

Understanding how bootstrap.servers works and why listing only a subset is acceptable deepens your knowledge of Kafka's client-server interactions. This understanding helps in making informed decisions when configuring Kafka clients for both performance and reliability.

**Code:**

```java
// Example Java configuration snippet
Properties props = new Properties();
props.put("bootstrap.servers", "broker1:9092,broker2:9092");
```

**Flow:**

1. Bootstrap Server List
2. All Brokers
3. Complete Connection Setup
4. Subset of Brokers
5. Initial Connection, Resilient toFailures
6. Connection Failure
7. Attempt Another BootstrapBroker

