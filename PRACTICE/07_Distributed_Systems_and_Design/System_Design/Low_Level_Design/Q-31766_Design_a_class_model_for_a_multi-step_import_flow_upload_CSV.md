---
type: "practice"
difficulty: "Intermediate"
---

# Design a class model for a multi-step import flow (upload CSV, validate, preview, commit) where each step's output feeds the next and users can abandon midway; how do you model the pipeline state and cleanup?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Compensating Transactions](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a class model for a multi-step import flow involving CSV upload, validation, preview, and commit, we need to ensure that each step's output feeds into the next while providing mechanisms to allow users to abandon the process at any point. The key challenge is managing the pipeline state across these steps and ensuring proper cleanup of resources.

The question is probing our understanding of state management in multi-step processes, error handling, resource management, and design patterns like Pipeline or Chain of Responsibility. Key assumptions include:

Steps will have distinct operations but should be reusable.
Each step can fail independently, requiring rollback mechanisms.
Users can navigate through steps without completing all, necessitating a clean way to represent partial completion.

We'll start by defining a PipelineStep class that encapsulates the behavior of each step and provides methods for transitioning between states. We’ll also introduce a PipelineState enum to manage the flow status and ensure proper cleanup.

#### First-Principles Solution
Define Pipeline State: Enumerate all possible pipeline states.
Create PipelineStep Class: Represent each step with its logic, state transitions, and output.
Implement Transition Logic: Ensure clean transitions between steps based on validation.
Cleanup Mechanism: Properly handle resource cleanup when a user aborts the process.
Step-by-Step Solution
FLOWCHART · MERMAID
classDiagram
    class PipelineState {
        +INITIALIZED: int
        +UPLOADED: int
        +VALIDATING: int
        +PREVIEWING: int
        +COMMITTED: int
        +ABORTED: int
        +toString(): string
    }

    class PipelineStep {
        +state: PipelineState
        +run(): void
        +nextStep(step: PipelineStep): void
        +previousStep(step: PipelineStep): void
        +cleanup(): void
    }

    class CSVUploadStep implements PipelineStep {
        +upload(file: File): void
        +validate(file: File): bool
        +saveIntermediateData(data: object): void
    }

    class ValidationStep implements PipelineStep {
        +validate(file: File): bool
        +saveErrors(errors: array): void
        +cleanup(): void
    }

    class PreviewStep implements PipelineStep {
        +preview(data: object): void
        +commitChanges(changes: object): void
        +cleanup(): void
    }

    CSVUploadStep -->| upload | ValidationStep
    ValidationStep -->| validate | PreviewStep
    PreviewStep -->| commitChanges, saveErrors | CommitStep

    PipelineState <|-- [INITIALIZED] : State
    PipelineState <|-- [UPLOADED] : State
    PipelineState <|-- [VALIDATING] : State
    PipelineState <|-- [PREVIEWING] : State
    PipelineState <|-- [COMMITTED] : State
    PipelineState <|-- [ABORTED] : State

    CSVUploadStep -[nextStep]-> ValidationStep
    ValidationStep -[nextStep]-> PreviewStep
Full Solution
PYTHON
class PipelineState:
    INITIALIZED = 1
    UPLOADED = 2
    VALIDATING = 3
    PREVIEWING = 4
    COMMITTED = 5
    ABORTED = 6

    @classmethod
    def toString(cls, state):
        return {
            cls.INITIALIZED: 'Initialized',
            cls.UPLOADED: 'Uploaded',
            cls.VALIDATING: 'Validating',
            cls.PREVIEWING: 'Previewing',
            cls.COMMITTED: 'Committed',
            cls.ABORTED: 'Aborted'
        }.get(state, 'Unknown')

class PipelineStep:
    def __init__(self):
        self.state = PipelineState.INITIALIZED

    def run(self):
        pass

    def nextStep(self, step):
        if self.state == PipelineState.UPLOADED and isinstance(step, ValidationStep):
            self.state = PipelineState.VALIDATING
            return True
        elif self.state == PipelineState.VALIDATING and isinstance(step, PreviewStep):
            self.state = PipelineState.PREVIEWING
            return True
        else:
            return False

    def previousStep(self, step):
        pass

    def cleanup(self):
        # Cleanup resources here
        pass

class CSVUploadStep(PipelineStep):
    def __init__(self):
        super().__init__()

    def upload(self, file):
        self.state = PipelineState.UPLOADED
        # Simulate saving intermediate data
        return True

    def validate(self, file):
        if self.state != PipelineState.UPLOADED:
            raise ValueError("Upload step not completed")
        # Validate CSV and save errors
        return True

class ValidationStep(PipelineStep):
    def __init__(self):
        super().__init__()

    def validate(self, file):
        if self.state != PipelineState.VALIDATING:
            raise ValueError("Validation step not reached")
        # Simulate validation logic
        return True

    def saveErrors(self, errors):
        # Save any errors encountered during validation
        pass

class PreviewStep(PipelineStep):
    def __init__(self):
        super().__init__()

    def preview(self, data):
        if self.state != PipelineState.PREVIEWING:
            raise ValueError("Preview step not reached")
        # Simulate preview logic
        return True

    def commitChanges(self, changes):
        # Commit any changes to the database or filesystem
        pass

# Example usage
upload_step = CSVUploadStep()
validate_step = ValidationStep()
preview_step = PreviewStep()

if upload_step.upload('file.csv'):
    if validate_step.validate('file.csv'):
        if preview_step.preview(validate_step.saveErrors([])):
            print("Committing changes")

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, we've designed a class model for managing a multi-step import flow by encapsulating each step's logic within PipelineStep classes. Each step can transition to the next only if the previous step has completed successfully. This design ensures proper state management and cleanup mechanisms are in place.

#### Common mistakes
include:

Not handling state transitions correctly, leading to invalid operations.
Forgetting to implement error handling and resource cleanup when steps fail or users abort the process.
Overcomplicating the logic by introducing unnecessary complexity without adding value.

For follow-ups, consider how you would handle concurrent user sessions or optimize performance for large datasets. Also, discuss potential security concerns, such as validating file content before processing it to prevent malicious uploads.

