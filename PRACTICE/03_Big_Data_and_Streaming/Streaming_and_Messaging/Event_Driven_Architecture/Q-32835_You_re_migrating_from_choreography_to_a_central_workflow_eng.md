---
type: "practice"
difficulty: "Expert"
---

# You're migrating from choreography to a central workflow engine for a 12-step fulfilment saga that's become impossible to reason about. Sequence the migration so each step is independently reversible and no in-flight saga is orphaned during the transition.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Choreography vs Orchestration](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Choreography_vs_Orchestration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

The key idea here is to sequence the migration from a choreography-based approach to a central workflow engine, ensuring that each step of the 12-step fulfilment saga can be independently reversible and no in-flight saga is orphaned during the transition. We will use a phased approach where we first create a new flow for each existing choreographed step before shutting down old systems.

#### Triaging the Question

The question involves migrating a complex, multi-step process (fulfilment saga) from a choreography-based design to a central workflow engine. The critical requirements are ensuring that each step can be independently reversible and no in-flight processes are orphaned during the transition. We need to sequence this migration carefully.

#### Assumptions
Current System: A set of 12 steps (Choreography).
Future System: Centralized Workflow Engine.
Reversibility Requirement: Each step must be reversible for independent rollbacks.
No Orphaned Saga: All in-flight sagas should not be lost during the transition.

#### Solution
Phase 1: Setup and Preparation
Define Reversible Steps:
Identify each choreographed step and define its reverse action if possible. For example, Step 3 might involve sending a confirmation email; the reverse could be canceling that email.
Implement Reverse Actions:
Ensure each reversible step has an associated reverse logic. This can be implemented as a separate function or method.
Phase 2: Create New Flow for Each Step
Step-by-Step Migration:
For each of the 12 steps, create a corresponding workflow task in the new engine.
Map existing step inputs and outputs to the new workflow tasks.
Ensure that each task can handle both forward and reverse operations.
Phase 3: Transition Data Structures
Data Migration Plan:
Plan how data will be migrated between old systems and the new central engine. This could involve batch jobs or real-time synchronization.
Test Data Flows:
Perform end-to-end testing to ensure that all data flows correctly through both systems.
Phase 4: Cut Over
Rolling Deployment:
Gradually shut down old choreography systems and enable the new workflow engine step by step.
Orphan Handling:
Implement mechanisms to handle in-flight sagas if some steps are still using the old system during the transition.
Phase 5: Validation and Monitoring
Post-Migration Testing:
Validate that all 12-step processes work correctly with both forward and reverse operations.
Monitoring:
Set up robust monitoring to detect any issues early in the new system.
Rollback Plan:
Have a detailed rollback plan if something goes wrong.

#### Closing Summary

By carefully sequencing the migration through these phases, we ensure that each step is reversible and no sagas are orphaned during the transition from choreography to a central workflow engine. This approach allows for smooth integration while maintaining operational integrity.

#### Common Mistakes/Follow-Ups
Not Implementing Reversibility: Failing to define or implement reverse actions can lead to unrecoverable states.
Ignoring Orphan Sagas: Not having a plan to handle in-flight sagas during the transition can result in data loss or inconsistencies.
Poor Testing: Skipping thorough testing and validation can introduce hidden bugs that only manifest post-deployment.

**Comparison:**

```
Yes
No
Yes
No
Send Order
Reverse?
CANCEL Order
Create Order
Process Payment
Reverse?
REFUND Payment
Process Payment
```

**Code:**

```python
# Example of reverse action implementation for Step 3
def send_confirmation_email(order_id):
    # Code to send email

def cancel_confirmation_email(order_id):
    # Code to cancel sent email
```

**Example input / output:**

| Step | Old System Action       | New System Task               | Reverse Action             |
|------|------------------------|------------------------------|----------------------------|
| 1    | send_order()           | create_order(order_id)        | cancel_order(order_id)     |
| 2    | process_payment()      | process_payment(order_id)     | refund_payment(order_id)   |
| ...  | ...                    | ...                          | ...                        |

**Data:**

```yaml
# Sample Data Migration Plan
steps:
  - step_name: "Send Confirmation Email"
    old_system_action: "send_confirmation_email()"
    new_system_task: "send_confirmation_email(order_id)"
    reverse_action: "cancel_confirmation_email(order_id)"
  - ...
```

**Flow:**

1. Start Migration
2. Define Reversible Steps
3. Implement Reverse Actions
4. Create New Flow for Each Step
5. Test Data Flows
6. Rolling Deployment (ShutDown Old Systems)
7. Orphan Handling Mechanisms
8. Post-Migration Testing
9. Monitoring and Rollback Plan

