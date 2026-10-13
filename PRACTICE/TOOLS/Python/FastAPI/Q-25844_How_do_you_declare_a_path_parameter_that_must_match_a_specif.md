---
type: "practice"
difficulty: "Basic"
---

# How do you declare a path parameter that must match a specific set of allowed values using an Enum in FastAPI?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Database Access and Tool Calls](<../../../../TOOLS/Python/FastAPI/Database_Access_and_Tool_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To declare a path parameter that must match a specific set of allowed values using an Enum in FastAPI, you first need to define your enum and then use it as a type annotation for the path parameter in your route definition. This approach ensures that only values from the specified enum are accepted, providing both robustness and type safety.

#### Triage
The question is asking how to restrict the input of a path parameter to specific allowed values using an Enum in FastAPI. It's probing our understanding of enums and type annotations in FastAPI. We'll assume you have basic knowledge of FastAPI and Python enums.

Step-by-Step Solution:

Define the Enum: Create an enum with the allowed values.
Use the Enum as a Type Annotation: Annotate the path parameter with this enum to restrict its possible inputs.
Handle Validation: Ensure that the input matches one of the enum's values; otherwise, return a 422 Unprocessable Entity error.

#### First-Principles Solution
Let's say we need to create an API endpoint for managing users where each user can have a specific status (e.g., NEW, ACTIVE, or INACTIVE). Here’s how you would implement this:

Define the Enum:

PYTHON
from fastapi import FastAPI, Path
from enum import Enum

class UserStatus(str, Enum):
    new = "new"
    active = "active"
    inactive = "inactive"

Create a Route with an Enum Path Parameter:

PYTHON
app = FastAPI()

@app.get("/users/{status}")
async def get_user_by_status(status: UserStatus):
    if status == UserStatus.new:
        return {"message": f"Handling new user requests"}
    elif status == UserStatus.active:
        return {"message": f"Handling active user requests"}
    else:
        return {"message": f"Handling inactive user requests"}

Validation and Error Handling: If an invalid value is provided, FastAPI will automatically handle the validation error and respond with a 422 Unprocessable Entity status.

#### Closing Summary
By defining a custom enum for your path parameter in FastAPI, you can enforce that only specific values are accepted, ensuring type safety and robust input handling. Common mistakes include not using enums or improperly annotating the path parameter, leading to incorrect validation behavior.

