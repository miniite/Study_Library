---
type: "practice"
difficulty: "Deep"
---

# A principal loop: design 'a class model where an operation must be both undoable and auditable' so the undo and the audit entry are derived from the same recorded intent; how do you avoid the classic bug where undo works but the audit log shows only the forward action?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Compensating Transactions](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: To design a class model that makes an operation both undoable and auditable, we need to ensure that every operation can be recorded in such a way that its undo and audit log entries are derived from the same intent. The key challenge is to avoid bugs where the undo operation works but the audit log shows only the forward action. This requires careful design of the state and behavior tracking.

#### Triage
The question asks us to create a class model for operations that need both undo and audit functionality. We must ensure these functionalities are consistent, meaning if an operation is recorded in the audit log, its inverse should correctly revert the system state when undone. Let's assume we have an application where user actions (like adding or removing items from a list) can be performed.

#### First-Principles Solution

State Representation: Each operation must maintain the state before and after the change. This ensures that both undo and audit operations can refer to this information.
Operation Composition: Operations should be composed in such a way that they are self-contained and provide their own undo behavior.
Audit Trail: Record detailed logs of each operation, including necessary metadata for audits.

Let's break down the solution step-by-step:

Define Operation Class:

Create an Operation class with methods to perform the action, record the state before and after, and generate audit entries.
Ensure that every operation can be reverted (undone) by its inverse operation.

Implement Undo Behavior:

Each Operation should have a method to undo the operation, which reverts the system state to the previous known good state.

Generate Audit Entries:

The audit entry must capture both the state before and after the operation for clarity and traceability.

Ensure Consistency:

During the creation of an Operation, record the current state as a baseline. When undoing, revert to this baseline.
Record detailed metadata in the audit log that includes who performed the action, when, and the resulting state changes.

Here's how we can implement these steps:

PYTHON
class Operation:
    def __init__(self):
        self.before_state = None
        self.after_state = None

    def perform(self, current_state):
        # Record the before state
        self.before_state = current_state.copy()

        # Perform the actual operation
        new_state = self._perform_action(current_state)
        self.after_state = new_state

        return new_state

    def _perform_action(self, current_state):
        raise NotImplementedError("Subclasses should implement this method")

    def undo(self, current_state=None):
        if not hasattr(self, 'before_state'):
            raise ValueError("Operation has no before state to revert from.")

        # Revert the system state to the before state
        return self.before_state

    def generate_audit_entry(self):
        return {
            "user": "John Doe",  # Example user
            "timestamp": "2023-10-05T14:48:00Z",
            "before_state": self.before_state,
            "after_state": self.after_state
        }

#### Closing Summary
and Common Mistakes/Follow-Ups:

In summary, designing an Operation class that ensures both undo and audit functionality involves maintaining consistent state records. The core idea is to ensure that every operation has a clear before and after state and that these states are used for generating detailed audit logs as well as for performing the inverse operations.

#### Common mistakes
include:

Not capturing the correct baseline state during Operation initialization.
Failing to properly reverse the state changes when undoing an operation.
Ignoring the need for comprehensive audit logs, leading to incomplete traceability and debugging capabilities.

For follow-ups, we might consider how this design scales with more complex operations or discuss how to integrate this into a larger system architecture for consistent behavior across multiple components.

