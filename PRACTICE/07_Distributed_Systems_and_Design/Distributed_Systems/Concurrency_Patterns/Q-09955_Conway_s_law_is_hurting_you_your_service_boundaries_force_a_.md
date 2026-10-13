---
type: "practice"
difficulty: "Expert"
---

# Conway's law is hurting you: your service boundaries force a distributed transaction with a nasty compensation race because two teams own the two halves. Would you re-org the teams, merge the services, or add coordination glue? Reason about the concurrency correctness AND the org cost of each.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Coordination Choices](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Distributed_Coordination_Choices.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

When faced with a distributed transaction and race conditions due to Conway's law, the decision on whether to re-organize teams, merge services, or add coordination glue hinges on evaluating both the technical and organizational impacts. Each option has its pros and cons, and the best choice depends on the specific context, such as system scale, complexity, and team dynamics.

#### Triage

The question revolves around managing concurrency correctness in a distributed system where two independent teams are responsible for different halves of a service that need to coordinate through transactions. This scenario often leads to race conditions, which can be challenging to debug and maintain. The key is to balance technical implementation with organizational realities.

#### Assumptions
Teams A and B own separate services but need to interact in a transactional manner.
There are race conditions due to the independent nature of these teams.
The system is mission-critical, requiring high reliability and minimal downtime.

#### First Principles Solution
Option 1: Re-Organize the Teams

#### Pros

Simplifies ownership and coordination within the organization.
Enhances communication between developers who need to understand each other's work better.
Allows for a unified codebase and shared responsibility, which can improve overall system reliability.

#### Cons

Potentially disrupts team dynamics and workflow.
May require significant retraining or restructuring of existing teams.
If not handled carefully, could lead to siloing in new boundaries if the organizational change is not fully integrated.
Option 2: Merge the Services

#### Pros

Centralizes logic and reduces race conditions by having a single service handle transactions.
Simplifies testing and deployment since there's no need for inter-team coordination during development.

#### Cons

Increases complexity and coupling within the monolithic system, potentially leading to more tightly coupled components that are harder to maintain.
May result in slower iteration cycles due to the combined size and scale of the codebase.
Could exacerbate Conway’s law if not managed carefully.
Option 3: Add Coordination Glue

#### Pros

Minimal disruption to existing teams and services, allowing them to continue working independently.
Can be implemented incrementally without significant rework or organizational changes.

#### Cons

Introduces additional complexity in the form of coordination logic, which can itself become a bottleneck if not properly designed.
Harder to ensure correctness as race conditions remain, requiring robust validation and monitoring mechanisms.
May lead to distributed transactions that are harder to debug and manage over time.

#### Closing Summary

Given the context where two teams own separate services that must interact in a transactional manner, adding coordination glue might be the most practical approach initially. This option allows for minimal disruption while maintaining existing team boundaries. However, if the complexity of race conditions becomes unmanageable, re-organizing the teams or merging the services could provide long-term benefits.

#### Common Mistakes
and Follow-Ups
Mistake: Ignoring the underlying Conway’s law issue by merely adding coordination glue.

#### Follow-Up
Discussing the implications on team morale and project timelines when merging services or re-organizing, considering factors like existing code quality and dependencies.

Would you consider any additional context that could impact this decision?

**Code:**

```
MERMAID
Copy
graph LR;
    A[Team A] -- Service1 --> B(Inter-service Coordination) --> C[Service2];
    A -- Service3 --> D[Compensation Logic];
    B -- Transaction Failure --> E[Mitigation Strategy];
```

**Example input / output:**

```text
# Example input and output for coordination logic
Input: TransactionRequest {service1Response, service2Status}
Output: TransactionOutcome {success: false, rollbackRequired: true, errorMessages: ["Service2 failed to confirm state"]}
Dry-run:
  - Team A requests Service1 from Team B's domain.
  - Service1 completes and responds successfully.
  - Team A then triggers a request for transactional confirmation with Service2.
  - Service2 confirms its state but fails in an unexpected way, triggering a rollback.
```

**Data:**

```yaml
# Example of service boundaries and coordination points
services:
  - name: Service1
    team: TeamA
  - name: Service2
    team: TeamB
  - name: Service3
    team: TeamA
compensation_logic:
  - service: Service2
    action_on_failure: RollbackTransaction
```

