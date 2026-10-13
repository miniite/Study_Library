---
type: "practice"
difficulty: "Intermediate"
---

# How does the producer's callback and Future help you know if a send succeeded?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pipeline Lag Monitoring](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Pipeline_Lag_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using Apache Kafka in your application, ensuring that messages are sent successfully is crucial. The Producer in Kafka provides mechanisms such as callbacks and Future to notify producers about the success or failure of message sends. In this answer, we'll delve into how these mechanisms work from first principles, discuss their benefits, and highlight common pitfalls.

The question probes your understanding of Kafka's producer model and its error handling capabilities. Specifically, it focuses on two key aspects:

How callbacks help in getting notified about send outcomes.
The role of Future in managing asynchronous operations.

#### Assumptions
include a basic familiarity with asynchronous programming concepts and an understanding that Kafka producers can operate in both synchronous and asynchronous modes.

Let's break down the solution step by step:

Step 1: Understanding Synchronous vs Asynchronous Modes

Kafka producers can operate in two primary modes:

Synchronous mode: The send method blocks until the message is successfully sent or an exception occurs.
Asynchronous mode: The send method returns a Future, which you can use to track the status of the send operation.
Step 2: Using Callbacks

Callbacks are functions that get executed once a specific event occurs. In the context of Kafka producers, callbacks are used to notify when a message has been successfully sent or an error has occurred.

Here’s how it works:

Message Sending: When you call send with a Callback, if the send is successful, the callback gets invoked with a RecordMetadata object.
Error Handling: If there's an error during sending (e.g., network issues), the callback receives an exception.
Step 3: Using Future

A Future in Kafka producers represents the result of an asynchronous operation and provides a way to manage the outcome without blocking. When you call send with a Callback, it returns a Future. This future can be used to check if the message has been sent successfully or to retrieve any errors that occurred.

Here’s a simple example in Java:

JAVA
ProducerRecord<String, String> record = new ProducerRecord<>("my-topic", "key", "value");
SendCallback sendCallback = (metadata, exception) -> {
    if (exception != null) {
        // Handle error case
    } else {
        // Handle successful case with metadata
    }
};

Future<RecordMetadata> future = producer.send(record, sendCallback);
Step 4: Managing Asynchronous Operations

Using Future allows you to manage multiple asynchronous operations more efficiently. You can:

Check if a Future has completed.
Retrieve the result of a successful operation or handle errors.
Cancel ongoing operations.

For example:

JAVA
Set<Future<RecordMetadata>> futures = new HashSet<>();
for (ProducerRecord<String, String> record : records) {
    Future<RecordMetadata> future = producer.send(record);
    futures.add(future);
}

// Wait for all sends to complete or check their status later
futures.forEach(f -> {
    try {
        RecordMetadata metadata = f.get(); // blocks until result is available
        System.out.println("Sent record: " + metadata.toString());
    } catch (InterruptedException | ExecutionException e) {
        Thread.currentThread().interrupt();
        throw new RuntimeException(e);
    }
});

#### Closing Summary
and Common Mistakes

Understanding callbacks and Future in Kafka producers allows you to effectively handle asynchronous operations. Key takeaways include:

Callbacks provide immediate feedback on the success or failure of send operations.
Futures allow you to manage multiple asynchronous sends efficiently, handling completion and errors gracefully.

#### Common mistakes
include:

Failing to implement error handling properly, leading to silent failures.
Not using futures to manage concurrent sends, resulting in inefficient resource utilization.

