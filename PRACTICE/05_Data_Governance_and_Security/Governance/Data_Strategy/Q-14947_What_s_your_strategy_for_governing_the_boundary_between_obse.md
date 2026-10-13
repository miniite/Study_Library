---
type: "practice"
difficulty: "Deep"
---

# What's your strategy for governing the boundary between observability data and business analytics data so incident metrics don't fork from BI's version of the same events?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Platform Telemetry](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Governing_Platform_Telemetry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In AIOps, effectively governing the boundary between observability data (OD) and business analytics data (BAD) is crucial to ensure that both streams remain aligned without duplicating or conflicting information, especially when it comes to incident metrics used by both teams. This strategy must balance visibility, accuracy, and efficiency across multiple domains.

The question is essentially probing your understanding of how to maintain a clear and consistent view of events between two critical functions: observability (often focused on technical infrastructure health) and business analytics (concerned with broader business operations). We need to ensure that when incidents are detected in the OD stream, they do not diverge from the BI perspective. This involves establishing robust data lineage, standardizing event definitions, and implementing a unified event model.

#### Triage

The interviewer is likely interested in your ability to design an integrated data strategy that ensures seamless communication between technical and business stakeholders while maintaining data integrity. Key assumptions include:

Both OD and BAD systems are mature enough to support integration.
There exists a shared understanding of critical events and metrics.
The organization has the necessary infrastructure for real-time data processing and storage.

#### First-Principles Solution

Data Model Alignment: Develop a unified event model that defines common event types, attributes, and their semantics across both OD and BAD systems. This ensures that an incident in one system is recognized as the same in the other.

Real-Time Data Synchronization: Use real-time data pipelines to synchronize critical events between OD and BAD. This prevents delays or discrepancies that could lead to divergent views of incidents.

Event Correlation: Implement advanced correlation techniques to match events across different systems based on metadata, timestamps, and event attributes. This helps in identifying the same incident in both streams.

Data Lineage Tracking: Maintain a clear lineage for each event to trace its origin and transformation history. This is crucial for debugging and ensuring data consistency.

Governance Policies: Establish governance policies that define who can modify events, how changes are propagated, and audit trails for any modifications.

Feedback Loops: Implement feedback mechanisms where the BI team can provide insights or corrections back to the OD team, ensuring a bidirectional flow of information.

#### Closing Summary

By aligning data models, synchronizing real-time events, correlating events across systems, tracking data lineage, and implementing robust governance policies, we can ensure that incident metrics in both observability and business analytics streams are consistent and aligned. This approach not only prevents divergence but also enhances overall operational visibility and efficiency.

#### Common Mistakes/Follow-Ups
Lack of Real-Time Synchronization: Not having a real-time pipeline can lead to delays in incident detection and resolution.
Insufficient Event Correlation: Relying solely on manual correlation can introduce errors and inconsistencies.
Poor Data Lineage Tracking: Without clear tracking, it’s difficult to trace data transformations and identify issues.
Ignoring Feedback Loops: Not allowing for bidirectional feedback can result in a one-way flow of information that may be incomplete or outdated.

**Code:**

```
MERMAID
Copy
graph LR
    A[OD Data Source] --> B[Real-Time Pipeline]
    B --> C[BAD System]
    C --> D[Incident Detection]
    D --> E[Event Correlation Engine]
    E --> F[Unified Event Model]
    F --> G[BI Dashboard]
```

**Example input / output:**

```text
flowDiagram
    participant OD as "OD System"
    participant BAD as "BAD System"

    OD ->> BAD: Synchronized Event (ApplicationError)
    BAD ->> OD: Correlation Check (Event Matched)
    OD ->> BAD: Unified Event Model Update
```

**Data:**

```yaml
# Unified Event Model
event_types:
  - type: ApplicationError
    attributes:
      - error_type
      - error_message

# Example Event in OD System
event_od:
  event_type: ApplicationError
  timestamp: 2023-10-01T14:30:00Z
  error_type: Critical
  error_message: "Database connection failed"

# Synchronized Event in BAD System
event_bad:
  event_type: ApplicationError
  timestamp: 2023-10-01T14:30:05Z
  error_type: Critical
  error_message: "Database connection failed"
```

