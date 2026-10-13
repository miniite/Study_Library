---
type: "practice"
difficulty: "Intermediate"
---

# Why would you turn on compression for a high-volume ingestion topic and which codec is a common default choice?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Producer Batching](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Producer_Batching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand when to use compression in Kafka for high-volume ingestion topics, we need to first grasp what Kafka does, its data transfer mechanisms, and how compression can optimize these processes. When dealing with a high-volume topic, enabling compression can significantly reduce the amount of data transferred over the network, thereby improving performance. The common codec used as the default is Snappy.

**Comparison:**

| Topic Parameters       | Values        |
|------------------------|---------------|
| Topic Name             | high_volume_topic |
| Partitions             | 10            |
| Replication Factor     | 3             |
| Compression Type       | Snappy        |

**Example input / output:**

```

Given a high-volume topic high_volume_topic with 10 partitions and replication factor of 3, the compression type is set to Snappy. This means that before data is sent over the network or written to disk, it will be compressed using Snappy.

MARKDOWN
Copy
| Topic Name     | Partitions | Replication Factor | Compression Type |
|----------------|------------|--------------------|------------------|
| high_volume_topic | 10         | 3                  | snappy           |
```

**Data:**

```json
{
  "topic": {
    "name": "high_volume_topic",
    "partitions": 10,
    "replication_factor": 3,
    "compression_type": "snappy"
  }
}
```

**Flow:**

1. Yes
2. No
3. Producer sends data
4. Data Volume?
5. Kafka compresses data usingSnappy
6. Compressed data sent overnetwork or written to disk
7. Data sent uncompressed
8. Improved performance due toreduced network bandwidthusage and storage space

