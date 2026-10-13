---
type: "practice"
difficulty: "Deep"
---

# Design the change-management process for a shared cross-cloud contract when a producer in cloud A wants a breaking change that impacts consumers in cloud B — who has veto and how is it phased?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-cloud Data Contracts](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Cross-cloud_Data_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In a multi-cloud environment, ensuring smooth and controlled changes to shared cross-cloud contracts requires a robust change-management process. The producer in Cloud A should initiate the process, but consumers in Cloud B must have veto power to protect their systems' integrity. This process involves detailed planning, communication, phased rollout, and rollback strategies.

#### Triaging the Question

The core of this question revolves around designing a systematic approach for implementing changes that are initiated by one party (producer) while ensuring another party (consumer) has the ability to prevent or delay changes if they pose risks. The key elements include initiation, approval, communication, phased rollout, and contingency planning.

#### Assumptions
Producer: A service in Cloud A that needs to make a breaking change.

#### Cons
umer: Services in Cloud B that rely on contracts with the producer.
Cross-Cloud Contracts: Interfaces or APIs defined between different services in separate cloud environments.
Veto Power: The consumer has the ability to reject or delay changes if they pose significant risks.

#### First-Principles Solution

Initiation and Documentation

The producer must document the proposed change, including its impact and justification.
This documentation is reviewed by a designated change management committee (CMT) that includes representatives from both Cloud A and Cloud B.

Approval Process

The CMT reviews the proposal to ensure it aligns with business goals and technical feasibility.
If approved, the producer proceeds; otherwise, the change is either revised or rejected.

Communication and Coordination

Once approved, the producer communicates the details of the upcoming change to all affected consumers in Cloud B.

#### Cons
umers are given a defined window (e.g., 1-2 weeks) to prepare for the change.

Phased Rollout

The change is implemented in phases to minimize disruption:
Phase 1: A small subset of services in Cloud B are updated first to test compatibility.
Phase 2: If Phase 1 is successful, a larger group of services in Cloud B get the update.
Phase 3: The final rollout to all services in Cloud B if no issues arise.

#### Cons
umer Veto Mechanism

#### Cons
umers can exercise their veto power during any phase:
During Phase 1: They might request more time for testing or ask for changes to be postponed entirely.
During Phases 2 and 3: They have the final say on whether the change should proceed.

Rollback Strategy

If a consumer exercises their veto, the producer must prepare a rollback plan:
Immediate steps to revert services back to the previous state if necessary.
Monitoring and support for affected services during recovery.

Post-Change Review

After the change is fully implemented, both parties conduct a post-mortem review:
Assessing whether the change was successful and meeting objectives.
Identifying any issues that arose and learning from them to improve future processes.

#### Closing Summary

The change-management process for cross-cloud contracts ensures smooth transitions while safeguarding against potential disruptions. By involving stakeholders, implementing phased rollouts, and providing veto power, we can balance innovation with stability.

#### Common Mistakes/Follow-Ups
Overlooking Consumer Input: Failing to adequately engage consumers in the change approval process.
Insufficient Testing: Skipping thorough testing phases or not accounting for rollback strategies.
Communication Gaps: Not maintaining clear and consistent communication channels between producers and consumers.
Lack of Flexibility: Failing to build in mechanisms for adapting to unexpected issues during rollout.
Triaging the Process

The process can be further refined by considering additional factors such as:

Impact Analysis Tools: Using tools to quantitatively assess the impact of changes before approval.
Automated Testing Frameworks: Implementing automated tests to ensure compatibility and reliability.
Service Level Agreements (SLAs): Defining SLAs that provide clear expectations and penalties for failure.

**Comparison:**

```
FLOWCHART · MERMAID
gantt
    title Multi-Cloud Change Management Process
    dateFormat YYYY-MM-DD
    section Initiation and Documentation
    Initiate Change: 2023-10-01, duration: 1
    section Approval
    CMT Reviews: 2023-10-02, duration: 5
    section Rollout Planning
    Communicate to B: 2023-10-07, duration: 1
    Start Phased R: 2023-10-08, duration: 30
    section Veto and Contingency
    Consumer Veto: optional, depends on phase progress
    Rollback Plan Created: depends on consumer veto decision
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Producer initiates change] --> B[Document Change];
    B --> C[CMT reviews proposal];
    C -- Approved?|Rejected? --> D[Proceed with changes|Reject change];
    D --> E[Communicate to Consumers];
    E --> F[Start Phased Rollout];
    F --> G[Phase 1: Test subset of services in Cloud B];
    G --> H[Phase 2: Roll out to larger group];
    H --> I[Phase 3: Full rollout];
    I --> J[Post-Change Review];
    G --> K[Veto Power (Consumer)] | L[Continue with phase] | M[Rollback Plan];
    K --> N[Adjust rollout plan based on veto];
```

**Example input / output:**

| Input            | Output                     |
|------------------|----------------------------|
| Initiate Change  | Documented Proposal        |
| CMT Reviews      | Approved/Rejected          |
| Communicate to B | Updated Consumers         |
| Start Phased R   | Phase 1 Testing           |
| Consumer Veto    | Rollback Plan Created     |
| Post-Change Rev  | Review Summary            |

**Data:**

```json
{
  "changeRequest": {
    "initiator": "Producer in Cloud A",
    "impactAnalysis": true,
    "approvalCommittee": ["Rep from Cloud A", "Rep from Cloud B"],
    "phases": [
      {"phaseName": "Phase 1 - Testing", "servicesUpdated": ["ServiceA"]},
      {"phaseName": "Phase 2 - Rollout", "servicesUpdated": ["ServiceB", "ServiceC"]},
      {"phaseName": "Phase 3 - Full Rollout", "servicesUpdated": ["All services"]}
    ]
  }
}
```

**Flow:**

1. Approved?|Rejected?
2. Initiate Change
3. Document Proposal
4. CMT Reviews Proposal
5. Proceed with Changes|RejectChange
6. Communicate to Consumers
7. Start Phased Rollout
8. Phase 1: Test Subset ofServices in Cloud B
9. J
10. Adjust Rollout Plan
11. H
12. Post-Change Review

