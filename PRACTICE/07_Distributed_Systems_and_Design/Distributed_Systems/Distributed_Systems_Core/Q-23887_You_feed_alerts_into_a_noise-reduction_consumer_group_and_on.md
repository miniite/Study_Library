---
type: "practice"
difficulty: "Intermediate"
---

# You feed alerts into a noise-reduction consumer group and one malformed alert repeatedly crashed the consumer, stalling a partition and delaying all alerts on it. Walk me through the poison-message DLQ strategy and how you'd replay after fixing the producer.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Dead Letter Queues and Outbox](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Dead_Letter_Queues_and_Outbox.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: In this scenario, we need to address a problematic alert that is causing consumer crashes by implementing a "poison message" dead-letter queue (DLQ) strategy. This involves detecting and isolating the faulty alerts, moving them to a DLQ, and then replaying them once the producer issue is fixed.

#### Triage
The core of this question revolves around understanding how to handle malformed messages in a distributed system, specifically within an Apache Kafka or similar messaging framework. We need to ensure that the consumer group can continue processing valid alerts without being stalled by problematic ones. The interviewer might be probing your knowledge on error handling mechanisms, message isolation strategies, and recovery processes.

#### First-Principles Solution

Identify the Poison Message: Use a poison message handler within the consumer logic to catch exceptions thrown when processing an alert.
Isolate the Faulty Alert: Once identified, move the problematic message to a DLQ.
Notify Stakeholders: Ensure there's a mechanism to notify developers or system administrators about the issue so that it can be fixed promptly.
Replay Mechanism: Set up a process to replay messages from the DLQ back into the main processing flow once the producer is fixed and validated.

Step-by-Step Solution:

#### Cons
umer Group Configuration:

Configure the consumer group to use a poison message handler. This can be done by implementing an onMessage callback or using a custom deserializer that can catch exceptions.

Dead-Letter Queue (DLQ):

Set up a DLQ with a different topic name, e.g., alerts-dlq.
Configure the consumer to send messages that throw exceptions directly to this DLQ.

Error Handling Logic:

JAVA
public class PoisonMessageHandler {
    private final Map<String, List<Alert>> dlqMessages = new ConcurrentHashMap<>();

    @KafkaListener(topics = "alerts", groupId = "consumer-group")
    public void consume(Alert alert) {
        try {
            // Process the alert
            process(alert);
        } catch (Exception e) {
            // Log and move to DLQ
            log.error("Error processing message: {}", alert, e);
            dlqMessages.computeIfAbsent(e.getClass().getName(), k -> new ArrayList<>()).add(alert);
            throw e; // Rethrow to ensure DLQ is triggered
        }
    }

    public void replayDlqMessages() {
        for (List<Alert> messages : dlqMessages.values()) {
            sendToTopic("alerts-dlq", messages);
        }
        dlqMessages.clear();
    }

    private void sendToTopic(String topic, List<Alert> alerts) {
        // Code to send alerts to the specified Kafka topic
    }
}

Notification Mechanism:

Integrate a notification system using tools like Slack or email to alert relevant team members when poison messages are detected.
JSON
{
    "alertLevel": "CRITICAL",
    "message": "Malformed alert detected at 2023-10-05 14:00. Please review and fix.",
    "timestamp": "2023-10-05T14:00:00Z"
}

Replay Mechanism:

After the producer issue is fixed, the consumer should be able to replay messages from the DLQ.
JAVA
@KafkaListener(topics = "alerts-dlq", groupId = "consumer-group")
public void replayConsume(Alert alert) {
    try {
        process(alert); // Process the message normally
    } catch (Exception e) {
        log.error("Error replaying message: {}", alert, e);
        throw e; // Rethrow to ensure re-processing
    }
}

Monitoring and Metrics:

Set up monitoring for the number of messages in the DLQ and any errors during the replay process.
Use Prometheus or similar tools to visualize these metrics.

#### Closing Summary
This approach ensures that problematic alerts are isolated, allowing other valid alerts to continue processing while issues are being addressed. It also provides a mechanism to notify stakeholders about the problem and ensure timely resolution. Common mistakes include failing to configure proper error handling, not integrating effective notification mechanisms, or neglecting to implement robust replay strategies.

#### Common Mistakes/Follow-Ups

Not considering edge cases where messages might be stuck in the DLQ indefinitely.
Failing to monitor DLQ metrics to ensure they remain low.
Overlooking the need for comprehensive testing of the replay mechanism.

**Comparison:**

```
Message	ID	Type	Timestamp
Alert	12345	CRITICAL	2023-10-05T14:00:00Z
DLQ	12345	CRITICAL	2023-10-05T14:00:00Z
```

**Code:**

```java
public class PoisonMessageHandler {
    private final Map<String, List<Alert>> dlqMessages = new ConcurrentHashMap<>();

    @KafkaListener(topics = "alerts", groupId = "consumer-group")
    public void consume(Alert alert) {
        try {
            process(alert);
        } catch (Exception e) {
            log.error("Error processing message: {}", alert, e);
            dlqMessages.computeIfAbsent(e.getClass().getName(), k -> new ArrayList<>()).add(alert);
            throw e;
        }
    }

    public void replayDlqMessages() {
        for (List<Alert> messages : dlqMessages.values()) {
            sendToTopic("alerts-dlq", messages);
        }
        dlqMessages.clear();
    }

    private void sendToTopic(String topic, List<Alert> alerts) {
        // Code to send alerts to the specified Kafka topic
    }
}

@KafkaListener(topics = "alerts-dlq", groupId = "consumer-group")
public void replayConsume(Alert alert) {
    try {
        process(alert);
    } catch (Exception e) {
        log.error("Error replaying message: {}", alert, e);
        throw e;
    }
}
```

**Example input / output:**

```text
Input:
- Alerts are being processed by a Kafka consumer group.
- A malformed alert crashes the consumer, causing a partition stall.

Output:
- Malformed alert is detected and moved to a DLQ topic 'alerts-dlq'.
- Consumer logs an error message about the issue.
- Notifications are sent to relevant stakeholders.
- Once producer fix is verified, replay process moves messages from DLQ back into main processing flow.
```

**Data:**

```json
{
    "alerts": [
        {"id": 12345, "type": "CRITICAL", "timestamp": "2023-10-05T14:00:00Z"},
        {"id": 67890, "type": "WARNING", "timestamp": "2023-10-05T14:15:00Z"}
    ],
    "dlqMessages": [
        {"id": 12345, "type": "CRITICAL", "timestamp": "2023-10-05T14:00:00Z"},
        {"id": 67890, "type": "WARNING", "timestamp": "2023-10-05T14:15:00Z"}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Consumer Receives Alert
4. Malformed?
5. Move to DLQ, Log Error
6. Process Alert Normally
7. Notify Stakeholders
8. Replay from DLQ if ProducerFixed

