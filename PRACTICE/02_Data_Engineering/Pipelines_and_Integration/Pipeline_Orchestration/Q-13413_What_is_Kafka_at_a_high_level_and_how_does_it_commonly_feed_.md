---
type: "practice"
difficulty: "Basic"
---

# What is Kafka at a high level and how does it commonly feed data into a Spark or Airflow pipeline?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Orchestration vs Processing](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Kafka is a distributed streaming platform that serves as a reliable message bus, enabling the efficient transfer of data between producers (like sensors, web applications) and consumers (like data processing pipelines). At its core, Kafka acts as a publish-subscribe messaging system with high throughput and low latency. In this context, Kafka is often used to feed real-time or near-real-time data into Spark or Airflow pipelines for further processing.

To understand how Kafka fits into these data pipelines, let's break down the components and their interactions.

**Comparison:**

```
Kafka Feature	Description
Reliability	Ensures messages are not lost through retries and acknowledgments.
Scalability	Can handle high throughput by adding more brokers or partitions.
Durability	Messages are stored on the filesystem, ensuring persistence.
Decoupling	Allows producers and consumers to scale independently without blocking each other.
```

**Code:**

```
MERMAID
Copy
graph LR;
    A[Producer] -->|Message| B[Kafka Broker]
    B -->|Topic| C[Consumer Group -> Spark or Airflow Pipeline]
```

**Example input / output:**

```

Here’s a basic example of how Kafka might be used to feed data into an Apache Airflow pipeline:

Producer: A simple application that generates and sends sensor readings.
Kafka Broker: Manages the topic where messages are published and consumed.
Consumer Group (Airflow Task): An Airflow task that consumes messages from the topic, processes them, and updates a database.

Example:

PYTHON
Copy
from kafka import KafkaProducer

# Producer example
producer = KafkaProducer(bootstrap_servers='localhost:9092')
sensor_reading = "12345"
producer.send('sensor-reading-topic', value=sensor_reading.encode())

Kafka ensures that messages are delivered reliably, even if some consumers fail or go down. This makes it ideal for integrating with robust data pipelines like Spark and Airflow.
```

**Flow:**

1. Data Source
2. Kafka Producer
3. Kafka Broker 1
4. Kafka Broker N
5. Kafka Topic -> Partition 1
6. Kafka Consumer Group -Airflow Task

