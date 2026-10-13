---
type: "practice"
difficulty: "Expert"
---

# How do you design consent propagation so that when a user revokes consent for marketing use, every feature and model that consumed that signal reflects it within SLA?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Consent Enforcement](<../../../../11_AI_and_ML/Responsible_AI/Privacy_and_PII/Consent_Enforcement.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that data privacy is respected and maintained in AI systems, particularly when users revoke consent for specific uses like marketing, we need a robust mechanism to propagate this change across all relevant features and models. This involves understanding the current state of affairs, identifying the dependencies, defining the scope, and implementing an efficient notification system.

#### Triage
Understanding the Question

The core of the question is about designing a consent revocation process that ensures immediate updates in feature and model implementations. The key is to ensure that any part of the system that relies on marketing data reflects this change within Service Level Agreement (SLA) limits, typically measured in minutes or hours.

#### Assumptions
Data Flow: Marketing data is used by multiple features and models.

#### Cons
ent Management System: A central system manages user consent across all applications.
Event-Driven Architecture: The system supports event-driven architecture to facilitate real-time updates.
SLA Compliance: The response time for updating models must meet predefined SLAs.
Step-by-Step Solution
1. Identify Dependencies

First, identify the features and models that use marketing data. This involves:

Mapping data flows: Determine which components are using this specific piece of data.
Defining impact zones: Understand how revoking consent will affect each component's output or behavior.
2. Centralized Consent Management System

Implement a centralized consent management system to track user consents and changes. Use a database schema like the following:

SQL
CREATE TABLE UserConsent (
    userId INT PRIMARY KEY,
    featureId INT,

#### cons
entType VARCHAR(50),
    status BOOLEAN,
    lastUpdated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
3. Event-Driven Architecture

Deploy an event-driven architecture where changes in user consent trigger events that are processed by relevant components.

Event Producer: The consent management system triggers a ConsentRevoked event when a user revokes marketing consent.
Event Consumers: Features and models subscribe to this event.
4. SLA Compliance

Ensure that the propagation of the event is as quick as possible:

Use message brokers like Kafka for reliable, real-time messaging.
Implement retry mechanisms and backoff strategies to handle transient issues.
5. Model Update Mechanism

For each feature or model:

Dry Run: Perform a dry run to simulate data changes without applying them until the SLA is met.
Real-Time Updates: Apply the updated consent status once the SLA is satisfied.
Grace Period: Allow a grace period for models that take longer to update, ensuring consistency.

#### Example
Workflow

User Revokes Consent:

User interface prompts user to revoke marketing consent.
Centralized consent management system marks status as false.

Event Triggered:

A ConsentRevoked event is generated and sent through Kafka.

Feature Notification:

Event is consumed by a feature that uses marketing data, triggering a dry run.
If the update can be applied immediately (e.g., within SLA), it is applied; otherwise, it waits until the grace period is over.

Model Update:

Models consume the updated consent status and reflect changes in their outputs or predictions.

#### Closing Summary

By implementing a centralized consent management system and an event-driven architecture, we can efficiently propagate changes to user consents across all relevant features and models while ensuring compliance with SLAs. This approach not only maintains data privacy but also ensures that our AI systems remain transparent and accountable to users.

#### Common Mistakes/Follow-Ups
Ignoring Edge Cases: Ensure the system handles cases where multiple consent revocations overlap.
Improper SLA Management: Define clear SLAs and monitor adherence to them.
Inadequate Logging/Debugging: Implement robust logging for tracing events and debugging issues.

**Comparison:**

| userId | featureId | consentType | status  | lastUpdated          |
|--------|----------|------------|---------|---------------------|
| 1234   | 5678     | marketing  | false   | 2023-10-10 14:00:00  |

**Code:**

```python
# Example event consumer in Python using Kafka
from kafka import KafkaConsumer

def handle_consent_revoke(event):
    feature_id = event['featureId']
    # Apply model update logic here
    print(f"Updating feature {feature_id} due to consent revocation")

consumer = KafkaConsumer('ConsentRevoked', bootstrap_servers=['localhost:9092'])
for message in consumer:
    handle_consent_revoke(message.value)
```

**Example input / output:**

```text
- **Input**: User revokes marketing consent.
- **Output**: `ConsentRevoked` event is triggered, and relevant features update their states.
```

**Data:**

```sql
CREATE TABLE UserConsent (
    userId INT PRIMARY KEY,
    featureId INT,
    consentType VARCHAR(50),
    status BOOLEAN,
    lastUpdated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

**Flow:**

1. User Revokes Consent
2. Consent Management System
3. Generate ConsentRevokedEvent
4. Kafka Broker
5. Feature X (Model Update)
6. Apply Model Update Logic
7. Notify User of Changes

