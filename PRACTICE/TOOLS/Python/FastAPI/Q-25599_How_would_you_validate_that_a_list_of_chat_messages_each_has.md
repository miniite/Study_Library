---
type: "practice"
difficulty: "Intermediate"
---

# How would you validate that a list of chat messages each has a valid role of system, user, or assistant in a FastAPI request?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pydantic Validation](<../../../../TOOLS/Python/FastAPI/Pydantic_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a list of chat messages in a FastAPI request contains only roles such as "system", "user", or "assistant," we need to implement input validation. This process involves checking each message's role to confirm it adheres to the expected values. We will use Pydantic models within FastAPI for this task, leveraging its powerful data validation features.

First, let’s break down the problem into steps:

Define a Pydantic model that includes the necessary fields.
Validate the incoming request against these constraints.
Handle potential errors gracefully and provide meaningful feedback to the user.

Now, we’ll go through each step in detail:

#### Triaging the Question

The core of this question is about validating input data within a FastAPI framework using Pydantic models. The interviewer might be probing your understanding of:

Data validation principles.
Using Pydantic models for robust schema enforcement.
Handling different levels of user and system errors gracefully.

Key Assumptions:

We are dealing with a POST request where the body contains a list of chat messages.
Each message has fields like role, content, etc., which we will focus on validating.

#### First-Principles Solution

We start by defining a Pydantic model to represent our ChatMessage. This model will include validation for the role field, ensuring it is one of "system", "user", or "assistant".

PYTHON
from pydantic import BaseModel, Field, validator
from typing import List

class ChatMessage(BaseModel):
    role: str = Field(..., description="Role must be 'system', 'user', or 'assistant'")
    content: str

    @validator('role')
    def valid_role(cls, v):
        valid_roles = ["system", "user", "assistant"]
        if v not in valid_roles:
            raise ValueError(f"Invalid role '{v}'")
        return v

Next, we define a route that accepts this list of ChatMessages and validates them.

PYTHON
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.post("/messages/")
async def receive_messages(messages: List[ChatMessage]):
    for message in messages:
        # Validate each message here; if any are invalid, raise an exception
        pass  # Placeholder for validation logic

    return {"status": "Messages received and validated successfully"}

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, we used Pydantic models to define a clear structure with role validation. This approach ensures that the data is not only structured correctly but also adheres to our specific business rules.

#### Common mistakes
include:

Not handling invalid inputs gracefully.
Overlooking edge cases where roles might be missing or incorrectly formatted.
Failing to provide informative error messages to users.

Potential follow-ups could explore more complex scenarios, such as nested structures within the ChatMessage model or integrating this validation with a larger FastAPI application.

**Code:**

```

from pydantic import BaseModel, Field, validator from typing import List

class ChatMessage(BaseModel): role: str = Field(..., description="Role must be 'system', 'user', or 'assistant'") content: str

CODE
Copy
@validator('role')
def valid_role(cls, v):
    valid_roles = ["system", "user", "assistant"]
    if v not in valid_roles:
        raise ValueError(f"Invalid role '{v}'")
    return v
CODE
Copy

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.post("/messages/")
async def receive_messages(messages: List[ChatMessage]):
    for message in messages:
        # Validate each message here; if any are invalid, raise an exception
        pass  # Placeholder for validation logic

    return {"status": "Messages received and validated successfully"}
```

