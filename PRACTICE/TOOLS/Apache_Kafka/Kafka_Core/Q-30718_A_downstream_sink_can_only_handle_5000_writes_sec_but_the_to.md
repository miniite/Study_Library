---
type: "practice"
difficulty: "Deep"
---

# A downstream sink can only handle 5000 writes/sec but the topic bursts to 50000/sec. Design the consumer flow-control so you neither drop data nor OOM.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Producer Batching](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Producer_Batching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of handling a high burst of messages (up to 50,000 writes/sec) by a downstream sink that can only handle 5,000 writes/sec without dropping data or running out of memory, we need to implement flow control and backpressure mechanisms. This ensures that the consumer processes messages at its maximum capacity while maintaining the integrity of the message stream.

#### Triaging the Question

The core issue is managing bursty traffic in a Kafka topic where the sink's processing rate is significantly lower than the peak input rate. The key points to consider are:

Flow Control: Ensuring that the producer can throttle the flow of data when the consumer cannot process at its full speed.
Backpressure: Implementing mechanisms for consumers to signal producers about their current load, allowing them to adjust their production rates accordingly.

#### Assumptions
and Context
We have a Kafka topic with varying write rates, peaking up to 50,000 messages/sec.
The downstream sink can process at most 5,000 messages/sec.
We must ensure that no data is dropped during bursts.
Memory usage should be controlled to prevent out-of-memory (OOM) errors.

#### First-Principles Solution
Produce-Side Flow Control: Implement a mechanism for the producer to slow down its write rate when it detects that the consumer is overwhelmed.

#### Cons
ume-Side Backpressure Handling: The consumer should be able to signal to the producer about its current processing state, enabling dynamic adjustment of message production rates.
Step-by-Step Solution

Producer Side:

Monitor the consumer's processing rate using a heartbeat mechanism or explicit backpressure signals.
When the consumer’s processing rate drops below a certain threshold, use Kafka’s built-in features to throttle the producer’s write rate.

#### Example
Use Kafka Streams API with max.in.flight.request.per.session and max.processing.delay to manage the flow.

#### Cons
umer Side:

Implement state management to track the current processing load.
Use Kafka's offsets or custom markers to signal to producers that they should slow down.

#### Example
Utilize Kafka’s SeekToCurrentBatch method to control message consumption based on backpressure signals from the producer.
Code Example
PYTHON
# Python example using Kafka Streams API

from kafka import KafkaConsumer, KafkaProducer
import time

def consumer():

#### cons
umer = KafkaConsumer('topic_name', bootstrap_servers=['localhost:9092'])

    # Track processing state
    last_processed_offset = 0

    for message in consumer:
        if (message.offset - last_processed_offset) > 1000:  # Adjust based on load
            print("Processing lag detected, throttling producer")
            time.sleep(1)  # Simulate backpressure
        else:
            process_message(message.value)
            last_processed_offset = message.offset

def producer():
    producer = KafkaProducer(bootstrap_servers=['localhost:9092'])

    while True:
        for i in range(5000):  # Simulated high write rate
            key = f"key_{i}".encode()
            value = f"value_{i}".encode()

            if should_produce():  # Backpressure check logic here
                producer.send('topic_name', key=key, value=value)

        time.sleep(1)  # Adjust based on consumer processing rate

def process_message(message):
    # Process message logic here
    pass

def should_produce():
    # Logic to determine if production should continue
    return True

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By implementing flow control at the producer side and backpressure handling on the consumer side, we can ensure that messages are processed without dropping or OOM errors.

#### Common Mistakes
Failing to implement proper state management for consumers.
Not using Kafka’s built-in features like max.in.flight.request.per.session effectively.
Overlooking dynamic adjustment of message production rates based on consumer load.

#### Follow-Ups
How would you handle long-term backpressure?
What other Kafka features could be used to manage bursty traffic?
How would you ensure the system remains resilient during high loads?

**Comparison:**

| Time | Produced Messages | Consumed Messages | Backpressure Signal |
|------|------------------|------------------|--------------------|
| 1s   | 50,000           | 5,000            | Throttle Producer   |
| 2s   | 4,000            | 5,000            | None               |
| 3s   | 2,500            | 5,000            | None               |

**Code:**

```python
# Example Python code using Kafka Streams API for flow control and backpressure handling.
```

**Example input / output:**

```text
Input:
- Producer sends messages at a high rate (50,000/sec)
- Consumer can handle only 5,000 messages/sec

Output:
- Consumer processes messages without dropping any.
- Producer throttles its write rate to match consumer capacity.
```

**Data:**

```json
{
  "topic_name": {
    "partitions": 10,
    "replication_factor": 3
  }
}
```

**Flow:**

1. Below threshold
2. Above threshold
3. Producer
4. Produce messages
5. Consumer processing rate
6. Throttle producer
7. Continue producing
8. Notify consumer to slow down

