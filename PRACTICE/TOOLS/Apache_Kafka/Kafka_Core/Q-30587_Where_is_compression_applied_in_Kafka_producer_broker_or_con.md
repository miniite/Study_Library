---
type: "practice"
difficulty: "Deep"
---

# Where is compression applied in Kafka — producer, broker, or consumer? Explain the flow.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Producer Batching](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Producer_Batching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Kafka, compression happens at the producer level before the message is sent to the brokers. This design decision ensures that data is compressed on the sender’s end, reducing the load on the network and storage. The flow of a Kafka topic involves producers sending messages, which are then stored in logs by the brokers.

To understand this deeply, let's break down the process step-by-step:

Producer: Messages are produced by applications that need to send data to Kafka.
Compression: Before being sent over the network or stored on disk, these messages undergo compression at the producer level. Commonly used codecs include Snappy, LZ4, and GZIP.
Network Transfer: Compressed messages are then transferred over the network to the brokers.
Broker Storage: Upon arrival at a broker, the messages are written directly into the log files without decompression.

Understanding this flow is crucial for optimizing Kafka clusters and managing storage efficiently. Compression can significantly reduce the amount of data stored, improving performance and reducing costs.

**Comparison:**

```
Producer	Network	Broker Storage
Compress	Transfer	No Decompress
```

**Code:**

```python
# Example producer configuration in Python with compression enabled using LZ4

from kafka import KafkaProducer

producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_compression_type='lz4'
)
```

**Flow:**

1. Yes
2. No
3. Producer
4. Is Compression Enabled?
5. Compress Message
6. Send to Broker
7. Broker Storage
8. Directly Send to Broker

