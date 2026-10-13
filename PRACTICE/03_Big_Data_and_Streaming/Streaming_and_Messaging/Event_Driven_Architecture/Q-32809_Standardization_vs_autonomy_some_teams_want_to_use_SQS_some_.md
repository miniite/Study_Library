---
type: "practice"
difficulty: "Deep"
---

# Standardization vs autonomy: some teams want to use SQS, some NATS, some Kafka, each with a real justification. How do you set a paved-road policy that permits deviation without fragmenting the org's operational and observability tooling?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Breaking Schema Changes](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/Breaking_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

In setting a paved-road policy for message queues in an organization, we aim to balance standardization with flexibility to accommodate diverse team needs while maintaining consistent operational and observability practices. This involves defining core standards that ensure interoperability and alignment across the organization, allowing for deviations where justified.

#### Triaging the Question

The question revolves around establishing a policy that supports both standardization and autonomy in message queue choices within an organization. Teams like Amazon SQS, NATS, and Apache Kafka each offer unique features and benefits, necessitating a nuanced approach to policymaking.

#### Assumptions
Teams have legitimate reasons for choosing specific queues: Each team's requirements might differ based on factors such as scalability, performance, or specific use cases.
Operational and observability tooling must be consistent: To ensure that the overall infrastructure remains manageable and predictable.
Paved road approach: Teams should follow standard practices unless there is a clear justification for deviating.

#### First-Principles Solution

Define Core Standards:

Message Protocol Compatibility: Ensure all message queues support common protocols like AMQP or MQTT to facilitate interoperability.
Event Schemas: Standardize event schemas and formats to ease integration across systems.
Key Metrics for Monitoring: Define essential metrics such as latency, throughput, and error rates that must be tracked consistently.

Deviation Policy:

Justification Requirement: Teams must provide a detailed justification for choosing an alternative queue technology (e.g., performance requirements, specific feature needs).
Approval Process: Implement a review process to evaluate justifications and ensure alignment with organizational goals.
Tooling Consistency: For approved deviations, ensure that the chosen system is integrated into existing observability tools.

Operational Practices:

Documentation: Maintain comprehensive documentation of standard practices, including setup procedures, configuration guidelines, and best practices.
Training Programs: Offer training sessions to ensure teams are well-versed in using standardized tools effectively.
Regular Audits: Conduct periodic audits to verify compliance with the policy.

Support Framework:

Centralized Support Team: Establish a support team that can provide guidance and assistance for both standard and non-standard queues.
Resource Allocation: Allocate necessary resources (e.g., training, tools) to support teams using different technologies.

#### Closing Summary

By defining core standards while allowing for reasonable deviations based on justifications, we can ensure an effective balance between consistency and flexibility. This approach supports team autonomy while maintaining a cohesive operational environment. Deviations must be carefully managed through established processes to avoid fragmentation and maintain overall system integrity.

#### Common Mistakes/Follow-Ups
Over-standardization: Failing to allow for necessary deviations can stifle innovation and limit the organization’s ability to adapt.
Lack of Justification Process: Without a clear process, teams might choose technologies haphazardly, leading to operational inconsistencies.
Insufficient Monitoring: Inconsistent monitoring practices can lead to blind spots in system performance and security.

#### Follow-Up
Questions:
How would you handle the integration challenges between different message queue systems?
What tools or techniques would you use to monitor and manage observability for a mixed environment?

**Code:**

```
MERMAID
Copy
graph LR;
    A[Define Core Standards] --> B[Deviation Policy];
    B --> C[Operational Practices];
    C --> D[Support Framework];
```

**Flow:**

1. Define Core Standards
2. Deviation Policy
3. Operational Practices
4. Support Framework

