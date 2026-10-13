---
type: "practice"
difficulty: "Expert"
---

# You're advising leadership on whether to adopt a vendor's 'exactly-once' managed service. Design the due-diligence questions and failure-injection tests that separate genuine guarantees from marketing, before signing a multi-year contract.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost of Exactly Once](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Cost_of_Exactly_Once.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When advising leadership on adopting an exactly-once managed service, it's crucial to ensure that the vendor’s claims match real-world performance and reliability. The key is to design due-diligence questions and failure-injection tests that can validate the semantics in a practical manner. Here’s how we'll approach this:

TL;DR

To determine if an 'exactly-once' managed service genuinely offers the required guarantees, we need to delve into its technical implementation, understand the underlying architecture, and perform rigorous testing through failure injection.

#### Triaging the Question

The core of the question is about assessing the vendor’s claims for exactly-once semantics. Specifically:

Vendor Claims: What guarantees are they making?
Technical Architecture: How do their systems ensure these guarantees?
Failure Cases: Are there edge cases or failure modes that might break the semantics?
Due-Diligence Questions

We need to ask detailed, technical questions to understand the vendor’s implementation:

Guarantees and Guarantees:

What does "exactly-once" mean in their context? (e.g., no duplicates, no losses)
Do they offer a strong ordering guarantee?
Are there any scenarios where they might fail to deliver messages?

Architectural Details:

How do they handle retries and redeliveries?
What mechanisms are in place for detecting duplicate or out-of-order messages?
Can you provide diagrams of their system architecture, particularly around message delivery, batching, and deduplication?

Error Handling and Recovery:

How do they ensure that errors during processing do not result in duplicate deliveries or message losses?
Do they use idempotent operations to prevent issues with retries?

Scalability and Performance:

Can their system scale under heavy load without violating exactly-once guarantees?
What are the performance characteristics of their deduplication and ordering mechanisms?

Monitoring and Logging:

How can we monitor and log message delivery failures to ensure compliance with exactly-once semantics?
Failure-Injection Tests

To validate these claims, we need to perform failure-injection tests that simulate common failure scenarios:

Message Loss and Redelivery:

Introduce simulated network partitions or crashes at various points in the system.
Verify that messages are not lost or delivered multiple times.

Duplicate Message Detection:

Send duplicate messages with varying intervals to ensure they are correctly detected and processed only once.

Ordering Guarantees:

Simulate concurrent message delivery and verify that order is preserved despite failures in the system.

Retry Logic:

Test retries under different failure conditions (e.g., network errors, processing failures) to ensure correct handling of duplicates.

Edge Cases:

Test scenarios where messages are processed out-of-order or where idempotent operations fail.
Verify that these edge cases do not break the exactly-once semantics.

#### Closing Summary
and Common Mistakes

By conducting thorough due-diligence through targeted questions and rigorous testing, we can validate whether a vendor's 'exactly-once' managed service genuinely meets our requirements. Common mistakes to avoid include:

Assuming marketing claims without technical validation.
Failing to test edge cases and failure modes.
Not understanding the underlying architecture and its implications on semantics.

#### Follow-Ups
Request detailed documentation of their implementation, including code snippets if possible.
Engage with multiple stakeholders within the vendor for a comprehensive understanding.

#### Cons
ider third-party reviews or benchmarks to supplement your findings.

**Comparison:**

| Test Case       | Scenario Description                 | Expected Outcome                      |
|-----------------|--------------------------------------|---------------------------------------|
| Network Partition | Introduce a simulated network split  | Messages are redelivered correctly    |
| Processing Failure   | Simulate an error in processing     | Message handling is idempotent         |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Message Producer] --> B[Broker];
    B --> C[Producer Acknowledgments];
    C --> D[Message Processor];
    D --> E[Consumer];
    E --> F[Consumer Acknowledgments];
    F --> G[Dead Letter Queue (DLQ)];
    G --> H[Retry Mechanism];
```

**Example input / output:**

| Test Case | Input Scenario                | Expected Output/Outcome                          |
|-----------|------------------------------|--------------------------------------------------|
| Network Partition | Producer sends a message, network partition occurs | Message should be redelivered to the processor and then consumer |
| Processing Failure   | Processor fails to process the message due to an error | Message should be sent to the DLQ for retry         |

**Data:**

```yaml
# Example failure scenario data for testing
failureScenario:
  - type: networkPartition
    description: Simulate a network partition between the broker and message processor.
  - type: processingFailure
    description: Introduce an error in the message processing logic to test retries.
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Producer
6. Is message critical?
7. Send with high priority
8. Queue normally
9. Broker
10. Message Processor
11. Consumer
12. Acknowledge Delivery
13. Is delivery confirmed?
14. End
15. Retry Logic
16. Dead Letter Queue (DLQ)
17. Log Failure
18. Notify Support

