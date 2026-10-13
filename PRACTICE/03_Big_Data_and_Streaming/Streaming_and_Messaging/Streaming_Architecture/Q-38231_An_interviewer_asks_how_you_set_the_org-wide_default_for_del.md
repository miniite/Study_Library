---
type: "practice"
difficulty: "Deep"
---

# An interviewer asks how you set the org-wide default for delivery semantics on the telemetry backbone. Would you make at-least-once the standard and let teams opt into exactly-once, or the reverse? Reason about the cost of the wrong default at 500-team scale.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cardinality Isolation](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Cardinality_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an org-wide context with a telemetry backbone at 500-team scale, setting the delivery semantics is a critical decision that impacts the reliability, performance, and overall system stability. The choice between making "at-least-once" (AO) the default or "exactly-once" (EO) the default hinges on understanding the trade-offs in terms of cost, complexity, and operational impact.

#### Triage
What's Being Probed

The interviewer is probing your understanding of delivery semantics, system reliability, and the implications of scalability. They are also assessing how you balance these factors to make a decision that aligns with organizational goals and technical constraints. Assumptions include:

Each team has specific needs in terms of data integrity.
There's no single "one-size-fits-all" solution due to varying use cases.
Cost is considered in both monetary and operational terms.

#### First Principles Solution
Delivering "Exactly-once" (EO) by Default

Reasoning:

Reliability: EO ensures that every message is processed exactly once, which is crucial for mission-critical systems where data loss or duplication can lead to severe consequences.

#### Cons
istency: In distributed systems, ensuring consistency is paramount. EO guarantees that the system state matches the real-world state, making it suitable for tasks like financial transactions and high-stakes decision-making processes.
Operational Complexity: While AO is simpler to implement and scale, managing duplicate processing can introduce significant complexity. Teams may need custom error handling logic, leading to increased operational overhead.

#### Implementation
Considerations:

Resource Utilization: EO requires more resources due to the additional checks needed for ensuring exactly-once delivery.
Latency: Slight increases in latency are expected as the system needs to handle idempotent operations and retries.
Error Handling: Teams need robust error handling mechanisms, which can add to development effort.
Delivering "At-least-once" (AO) by Default

Reasoning:

Simplicity: AO is easier to implement and scale as it doesn't require the overhead of ensuring exactly once.
Performance: Lower latency and simpler error handling reduce operational complexity, allowing for faster deployments and quicker iterations.
Use Case Flexibility: AO is suitable for scenarios where data duplication can be managed by business logic or where processing speed outweighs the need for absolute consistency.

#### Implementation
Considerations:

Data Integrity: Teams must implement custom logic to handle duplicates, which can lead to errors if not done correctly.
Scalability: While AO scales well, teams may need to invest in additional resources and processes to manage duplicate data effectively.
Operational Overhead: Managing duplicates introduces operational overhead that can be burdensome for some use cases.

#### Closing Summary

Given the scale of 500 teams, making exactly-once (EO) the default is a more prudent choice due to its inherent reliability benefits. However, this decision must come with clear communication and training to ensure all teams understand the implications and have the necessary tools to handle the additional complexity. Conversely, opting for at-least-once (AO) by default might be suitable if the primary focus is on reducing operational overhead and improving development speed.

#### Common Mistakes/Follow-ups
Overlooking Team-Specific Needs: Assuming all teams will use exactly-once semantics without considering their specific requirements.
Neglecting Operational Overhead: Failing to account for the increased resource utilization and complexity when implementing exactly-once semantics.
Ignoring Custom Error Handling: Teams may struggle with custom error handling if AO is chosen by default, leading to data inconsistencies.

#### Triage
Revisited

The key takeaway is that setting the wrong default can lead to significant operational issues at scale. The decision should be made based on a balanced evaluation of reliability and performance needs across all teams.

**Code:**

```
MERMAID
Copy
graph TD;
    A[AO Default] --> B1[Custom Error Handling];
    A --> B2[Increased Operational Overhead];
    E[EO Default] --> F1[Resource Utilization];
    E --> F2[Latency Increase];
```

**Example input / output:**

```text
Input: Set default delivery semantics for telemetry backbone.
Output:
- AO Default:
  - Team Finance: Custom error handling implementation required.
  - Team Operations: No custom logic needed, simpler operations.

- EO Default:
  - Team Finance: Increased resource utilization and operational complexity.
  - Team Operations: Lower latency and faster deployments.
```

**Data:**

```json
{
  "teams": [
    {
      "name": "Finance",
      "deliverySemantics": "ExactlyOnce"
    },
    {
      "name": "Operations",
      "deliverySemantics": "AtLeastOnce"
    }
  ]
}
```

