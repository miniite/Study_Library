---
type: "practice"
difficulty: "Intermediate"
---

# What is long polling in SQS and why use it over short polling?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Visibility Timeouts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Visibility_Timeouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Long polling in Amazon Simple Queue Service (SQS) is an optimization technique where a client sends a request to the SQS endpoint, and the server holds that connection open until there are messages available or the timeout period is reached. This approach allows for more efficient use of resources by reducing the number of unnecessary polls.

#### Triage

The question asks about long polling in SQS and why it might be preferred over short polling. Long polling is a technique used to optimize message retrieval from queues, while short polling involves frequently checking if there are any new messages available. The interviewer is probing your understanding of message queuing concepts, the trade-offs between different polling strategies, and their implications on system performance.

#### First-Principles Solution

Short polling involves sending a request to check for messages every few seconds, even when no messages are present. This approach can lead to high resource usage because it frequently makes unnecessary network calls. Long polling, by contrast, keeps the connection open until a message is available or until a timeout occurs. Here’s a step-by-step breakdown of why long polling is useful:

Resource Efficiency: By waiting for a message before making a request, long polling significantly reduces the number of unnecessary connections to the SQS endpoint. This can drastically lower the rate of network requests and improve overall system performance.
Reduced Latency: When messages are infrequent or unpredictable, short polling can lead to significant delays in processing because it might not detect new messages immediately. Long polling ensures that once a message is available, it gets processed quickly without waiting for the next scheduled poll.
Scales Better with Message Volume: In scenarios where the number of messages fluctuates greatly, long polling can handle bursts more gracefully than short polling. It avoids the overhead of frequent polls during periods of low activity and ensures timely processing when there are many incoming messages.

#### Closing Summary

Long polling in SQS is a valuable technique that optimizes message retrieval by reducing unnecessary network calls and improving response times. While it adds some latency to individual requests, its overall benefits in terms of resource efficiency and better handling of variable message volumes make it a preferred choice over short polling in many use cases.

#### Common Mistakes/Follow-Ups
Misunderstanding the Trade-offs: Some engineers might think that long polling is always better because it reduces network calls. However, in scenarios with frequent messages, short polling can still be more efficient.
Ignoring Network Latency: Long polling introduces additional latency due to the wait period before a message is fetched. This must be considered when real-time requirements are stringent.
Overlooking Cost Implications: While long polling reduces network calls, it might not always result in cost savings, especially if there are significant idle periods where no messages are available.

For follow-ups, you could discuss scenarios where short polling would be more advantageous or explore how to combine both strategies for optimized performance based on specific use cases.

**Comparison:**

```
Polling Type	Connection Frequency	Resource Usage	Latency
Short Polling	Every few seconds	High	Low
Long Polling	Until message or timeout	Low	Higher (due to wait time)
```

**Code:**

```python
# Example of long polling in SQS (Pseudo-code)
def receive_message(sqs_client):
    while True:
        response = sqs_client.receive_message(
            QueueUrl='https://queue-url.amazonaws.com/your-queue',
            WaitTimeSeconds=20  # Long Polling timeout
        )
        if 'Messages' in response:
            return [msg['Body'] for msg in response['Messages']]
```

**Example input / output:**

```text
Input:
- QueueUrl = 'https://queue-url.amazonaws.com/your-queue'
- WaitTimeSeconds = 20

Output:
- Messages = ['Message Body 1', 'Message Body 2']
```

**Data:**

```json
{
    "QueueUrl": "https://queue-url.amazonaws.com/your-queue",
    "WaitTimeSeconds": 20,
    "Messages": [
        {"ReceiptHandle": "handle1", "Body": "Message Body 1"},
        {"ReceiptHandle": "handle2", "Body": "Message Body 2"}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Receive Message Request
4. Messages Available?
5. Fetch Messages
6. Wait for Timeout

