---
type: "practice"
difficulty: "Deep"
---

# A failed recurring charge triggers dunning; how would you design the retry-and-notify state machine so it's observable, doesn't double-charge, and can be paused for a customer in a dispute?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Compensating Transactions](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To design a robust retry-and-notify state machine that handles failed recurring charges, we need to ensure observability, prevent double-charging, and handle disputes. The core idea is to model states with clear transitions, use event-driven notifications for each step, and allow pausing the state machine.

#### Triage
What's Being Probed

This question tests your understanding of low-level design principles in backend systems, specifically focusing on:

State Machine Design: How you structure different states and transitions.
Concurrency Control: Preventing double-charges when retries occur.
Observability and Tracing: Ensuring the system’s behavior can be monitored and debugged.
Dispute Management: Handling scenarios where a customer is in dispute and requires special handling.

#### Assumptions
A failed charge triggers a retry mechanism after an initial delay.
The state machine should notify relevant parties (e.g., customers, support teams) at key points.
Retries are limited to avoid infinite loops.

#### First-Principles Solution
1. State Definitions and Transitions

We define the following states for our state machine:

Initial: New recurring charge created but not yet attempted.
Pending Retry: Charge failed initially, awaiting retry attempt.
Retry Attempted: A retry is currently in progress.
Succeeded: The charge was successfully processed after retries.
Failed: The charge cannot be processed even after retries.

The transitions between states are as follows:

From Initial to Pending Retry: On initial failed charge.
From Pending Retry to Retry Attempted: When the retry interval expires and a new attempt is made.
From Retry Attempted to either Succeeded or Failed: Based on the outcome of the retry.
2. Observability

To ensure observability, each state transition should be logged with relevant metadata (e.g., charge ID, customer ID, timestamp). Notifications can be triggered at specific points:

Notify support team when a charge fails initially.
Inform customers via email or SMS about failed attempts and pending retries.
Log detailed information for auditing purposes.
3. Concurrency Control

To prevent double-charging:

Use an atomic operation to update the state machine’s internal state.
Ensure that each retry attempt checks if the charge is already marked as succeeded or failed before proceeding.
Implement a rate-limiting mechanism on retries to avoid excessive attempts.
4. Dispute Management

For customers in disputes, we need additional handling:

Pause all retries for the customer until the dispute is resolved.
Track disputes separately and provide an interface to manually resume or modify charges.

#### Implementation
Details
FLOWCHART · MERMAID
graph TD;
    A[Initial] --> B(Pending Retry);
    B --> C(Retry Attempted) { "Retry Interval Expires" };
    C --> D(Succeeded);
    C --> E(Failed) { "Charge Succeeds After Retries" };
    E --> F[Failed];
    B --> F;

#### Example
Worked Input-Output
JSON
{
  "chargeId": "12345",
  "customerId": "67890",
  "state": "Initial",
  "retryAttempts": 0,
  "disputeStatus": null,
  "lastRetryAttemptTime": null,
  "nextRetryAttemptTime": null
}
Initial: {"chargeId": "12345", "customerId": "67890", "state": "Pending Retry", "retryAttempts": 1, "disputeStatus": null}
Pending Retry: {"chargeId": "12345", "customerId": "67890", "state": "Retry Attempted", "retryAttempts": 1, "disputeStatus": null}
Succeeded: {"chargeId": "12345", "customerId": "67890", "state": "Succeeded", "retryAttempts": 1, "disputeStatus": null}
Failed: {"chargeId": "12345", "customerId": "67890", "state": "Failed", "retryAttempts": 1, "disputeStatus": null}

#### Closing Summary

The design involves a state machine with well-defined states and transitions. Observability is ensured through logs and notifications. Concurrency control prevents double-charging by using atomic operations and rate limiting. Dispute handling allows pausing retries and managing disputes separately.

#### Common Mistakes/Follow-Ups
Ignoring concurrency: Not properly synchronizing retries can lead to double-charges.
Lack of observability: Without logs and notifications, debugging issues becomes difficult.
Insufficient dispute management: Customers in disputes might be charged unnecessarily if not handled correctly.

**Comparison:**

```
State	Next Action	Outcome
Initial	Attempt Retry	Pending Retry
Pending Retry	Wait for retry interval	Retry Attempted
Retry Attempted	Charge Succeeded	Succeeded
Retry Attempted	Charge Failed	Failed
```

**Code:**

```python
# Example code snippet for state transitions
class ChargeStateMachine:
    def __init__(self, charge_id, customer_id):
        self.charge_id = charge_id
        self.customer_id = customer_id
        self.state = "Initial"
        self.retry_attempts = 0
        self.dispute_status = None

    def handle_retry(self):
        if self.state == "Pending Retry":
            # Logic to attempt retry, update state, and trigger notifications
            pass
        elif self.state == "Retry Attempted":
            # Logic to check success or failure of retry, update state, and notify
            pass
```

**Example input / output:**

```text
{
  "chargeId": "12345",
  "customerId": "67890",
  "state": "Pending Retry",
  "retryAttempts": 1,
  "disputeStatus": null,
  "lastRetryAttemptTime": "2023-10-01T12:00:00Z",
  "nextRetryAttemptTime": "2023-10-05T12:00:00Z"
}
```

**Data:**

```json
{
  "chargeId": "12345",
  "customerId": "67890",
  "state": "Initial",
  "retryAttempts": 0,
  "disputeStatus": null,
  "lastRetryAttemptTime": null,
  "nextRetryAttemptTime": null
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Initial] --> B(Pending Retry);
4. B --> C(Retry Attempted) { "Retry Interval Expires" };
5. C --> D(Succeeded);
6. C --> E(Failed) { "Charge Succeeds After Retries" };
7. E --> F[Failed];
8. B --> F;

