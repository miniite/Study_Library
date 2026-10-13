---
type: "practice"
difficulty: "Deep"
---

# How do you safely roll out a change to a Pydantic request model that adds a required field without breaking older clients still sending the old payload?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pydantic Validation](<../../../../TOOLS/Python/Python_Language/Pydantic_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with changes in a Pydantic request model, especially when adding a new required field, it's crucial to ensure backward compatibility and avoid breaking existing client applications that haven't been updated yet. This scenario involves a thoughtful approach to handling the transition period without compromising data integrity or application stability.

#### Triaging the Question

The core of this question is about safely rolling out an update in a Pydantic model that introduces a new required field, ensuring that older clients continue to function correctly. The interviewer is probing your understanding of how to manage changes in schema definitions and handle partial requests gracefully. Key assumptions include:

Existing clients are not updated to use the new field.
Data consistency must be maintained during the transition.

#### First Principles Solution

To address this problem, we can leverage Pydantic's flexibility through data parsing and validation. The approach involves creating a custom validator function that checks for the presence of the new required field. If the field is missing, it will set a default value or raise an appropriate error. This method ensures that older clients can continue to send their payloads without issue while still supporting the new requirements.

Step-by-Step Solution
Define the New Pydantic Model:
Introduce the new required field in your Pydantic model.
Custom Validator Function:
Implement a validator function that checks for the presence of the new field and handles its absence appropriately.
Update Client Code Gradually:
Guide clients to update their code incrementally.

#### Example
Code

Here's how you might implement this in Python using Pydantic:

PYTHON
from pydantic import BaseModel, validator

class OldRequestModel(BaseModel):
    field1: str
    field2: int

# Introduce the new required field
class NewRequestModel(OldRequestModel):
    # Add a new required field
    new_field: float

    @validator('new_field', allow_reuse=True)
    def validate_new_field(cls, v):
        """Ensure 'new_field' is present and valid."""
        if cls.__parent__.__name__ == 'NewRequestModel':
            return v  # Return the value as it's required
        else:
            raise ValueError("This field is required for new clients")

#### Closing Summary
+ Common Mistakes/Follow-Ups

By using a custom validator, you ensure that only new clients sending data with new_field will be affected. This approach keeps older clients functioning correctly while preparing the infrastructure for future updates. Common mistakes include:

Not handling missing fields gracefully in the validator.
Forgetting to update client documentation and guidance.
Common Follow-Ups
How do you handle large-scale rollouts of such changes?
What about database migrations related to this change?
Can you discuss any performance implications?

