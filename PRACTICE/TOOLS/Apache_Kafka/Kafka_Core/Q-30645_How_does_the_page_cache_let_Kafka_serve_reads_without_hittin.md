---
type: "practice"
difficulty: "Deep"
---

# How does the page cache let Kafka serve reads without hitting disk, and why is JVM heap NOT where you want Kafka's data cached?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Claim Check Pattern](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Claim_Check_Pattern.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how Kafka leverages page caching to serve reads efficiently without hitting disk, we need to delve into the architecture of Kafka, particularly its storage mechanisms and how they interact with the operating system (OS). This involves understanding that Kafka primarily stores data on disk in log segments but uses OS page caching for performance optimization. The JVM heap is not an ideal place for storing Kafka's data due to several constraints related to memory management and garbage collection.

Kafka writes logs to disk efficiently using a dedicated thread, which ensures durability while maintaining high write throughput. However, when reading from these logs, Kafka can benefit significantly from the OS page cache. When the operating system caches file data in its page cache, subsequent reads can be served directly from the cache without hitting the disk, leading to faster read performance.

The key idea here is that the OS manages caching more efficiently than we would manage it ourselves using the JVM heap. The OS optimizes for caching by considering factors like memory pressure, access patterns, and eviction policies, which are challenging to replicate manually in a Java application.

**Comparison:**

```
Cache Mechanism	JVM Heap vs. OS Page Cache
Memory Management	Complex, Garbage Collected
Performance	Slower due to GC overhead
Control	Limited control
```

**Code:**

```java
// Example of Kafka consumer configuration that touches on caching mechanisms indirectly
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("group.id", "my-group");
props.put("enable.auto.commit", "true");
props.put("auto.commit.interval.ms", "1000");
props.put("session.timeout.ms", "30000");
props.put("key.deserializer", StringDeserializer.class.getName());
props.put("value.deserializer", StringDeserializer.class.getName());

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("my-topic"));
```

**Example input / output:**

```text
Input:
- A Kafka topic with log segments stored on disk

Output:
- Read operations served from the OS page cache, reducing disk I/O and improving read performance
```

**Flow:**

1. Miss
2. Hit
3. Write Log
4. OS Page Cache
5. Read Request
6. Kafka Disk Read
7. Cache Data in OS Page Cache
8. Serve from OS Page Cache

