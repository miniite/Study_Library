---
type: "practice"
difficulty: "Basic"
---

# How would you use kafka-console-producer and kafka-console-consumer to quickly debug whether messages are flowing?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pipeline Lag Monitoring](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Pipeline_Lag_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To debug message flow using kafka-console-producer and kafka-console-consumer, we'll leverage these tools to directly interact with a Kafka topic, allowing us to send and receive messages in real-time. This approach is particularly useful during development or troubleshooting phases when you need quick insights into the messaging pipeline.

The question probes your understanding of how to practically use command-line utilities for debugging purposes. We’ll assume that the Kafka cluster is up and running, and we have access to the topic where our messages should be flowing.

Firstly, let's break down the steps:

Identify the Topic: Determine which Kafka topic contains the messages you want to inspect.
Start kafka-console-producer: This utility will allow us to send messages to the specified topic.
Start kafka-console-consumer with appropriate settings: This utility will let us consume and display messages from the topic in real-time.

Now, we'll walk through setting up these tools step-by-step.

Identify the Topic:

Let's assume the topic is named messages-topic.

Start kafka-console-producer:

Open a terminal or command prompt and run:
BASH
kafka-console-producer.sh --broker-list <brokers> --topic messages-topic
Replace <brokers> with the list of Kafka broker addresses, e.g., localhost:9092.

Start kafka-console-consumer:

Open another terminal or command prompt and run:
BASH
kafka-console-consumer.sh --bootstrap-server <brokers> --topic messages-topic --from-beginning
Again, replace <brokers> with the list of Kafka broker addresses. The --from-beginning option ensures that you can see all messages from the start of the topic.

Send Messages:

In the kafka-console-producer terminal, type your message and press Enter to send it.
You should immediately see the same message appear in the kafka-console-consumer terminal, confirming that the message has been successfully delivered and consumed.

Validate Message Flow:

By observing the messages in both terminals, you can quickly identify any discrepancies or issues in the flow. For example, if a message is not appearing in the consumer, it might indicate a problem with the producer, network, or broker configuration.

Stop When Done:

Press Ctrl+C in each terminal to stop the producers and consumers.

