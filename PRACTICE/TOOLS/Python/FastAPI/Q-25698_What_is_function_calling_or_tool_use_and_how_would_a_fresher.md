---
type: "practice"
difficulty: "Intermediate"
---

# What is function calling or tool use, and how would a fresher expose a simple calculator tool to an LLM via a FastAPI endpoint?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ LLM Endpoints in FastAPI](<../../../../TOOLS/Python/FastAPI/LLM_Endpoints_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are tasked with exposing a simple calculator tool via a FastAPI endpoint that can be utilized by an LLM (Language Model). The core idea is to create a RESTful API where users can send HTTP requests with arithmetic operations and receive the results. For a fresher in the field of AI engineering, it's essential to understand how to set up this functionality from first principles.

#### Triage
What Are We Doing?

The question asks us to implement a tool use or function calling scenario using FastAPI. Specifically, we need to:

Define an API endpoint for arithmetic operations.
Handle incoming requests and parse the operation data.
Perform the calculation and return the result.
Ensure robust error handling.

#### Assumptions
We assume that the LLM will send a JSON request with the arithmetic operation details, such as operation (like 'add', 'subtract', etc.), a, and b.
The API will support basic arithmetic operations like addition, subtraction, multiplication, and division.
Error handling includes cases where invalid inputs or unsupported operations are provided.
Step-by-Step Solution

Set Up the FastAPI Environment:

Install FastAPI using pip:
BASH
pip install fastapi[all]
Use Uvicorn as the ASGI server to run the application.

Define the Calculator Functions:

We will define simple functions for each arithmetic operation.

Create an API Endpoint:

Implement a FastAPI endpoint that accepts POST requests with JSON payloads containing the operation and operands.

Handle Requests and Perform Calculations:

Parse the incoming request.
Validate the input data.
Call the appropriate calculator function based on the operation.
Return the result or an error message if necessary.
Code Implementation
PYTHON
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

# Define a Pydantic model for request body
class CalculationRequest(BaseModel):
    operation: str
    a: float
    b: float

# Calculator functions
def add(a: float, b: float) -> float:
    return a + b

def subtract(a: float, b: float) -> float:
    return a - b

def multiply(a: float, b: float) -> float:
    return a * b

def divide(a: float, b: float) -> float:
    if b == 0:
        raise HTTPException(status_code=422, detail="Division by zero is not allowed")
    return a / b

@app.post("/calculate/")
async def calculate(request: CalculationRequest):
    operation = request.operation
    a = request.a
    b = request.b

    # Perform the calculation based on the operation
    if operation == "add":
        result = add(a, b)
    elif operation == "subtract":
        result = subtract(a, b)
    elif operation == "multiply":
        result = multiply(a, b)
    elif operation == "divide":
        result = divide(a, b)
    else:
        raise HTTPException(status_code=422, detail="Unsupported operation")

    return {"result": result}

# Example request JSON
# {
#     "operation": "add",
#     "a": 5,
#     "b": 3
# }

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, we have created a FastAPI application that exposes an arithmetic calculator tool via a POST endpoint. The solution involves setting up the environment, defining calculator functions, creating a Pydantic model for request validation, and handling different operation scenarios.

#### Common mistakes
to avoid include:

Not validating input data thoroughly.
Failing to handle edge cases like division by zero.
Missing proper error handling mechanisms.

For follow-ups, one might ask about optimizing performance for large-scale use or integrating with real-world APIs. Alternatively, the interviewer could probe deeper into security measures and rate limiting.

**Comparison:**

```
Operation	Description
add	Adds two numbers.
subtract	Subtracts the second number from the first.
multiply	Multiplies two numbers.
divide	Divides the first number by the second, handling division by zero errors.
```

**Code:**

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class CalculationRequest(BaseModel):
    operation: str
    a: float
    b: float

def add(a: float, b: float) -> float:
    return a + b

def subtract(a: float, b: float) -> float:
    return a - b

def multiply(a: float, b: float) -> float:
    return a * b

def divide(a: float, b: float) -> float:
    if b == 0:
        raise HTTPException(status_code=422, detail="Division by zero is not allowed")
    return a / b

@app.post("/calculate/")
async def calculate(request: CalculationRequest):
    operation = request.operation
    a = request.a
    b = request.b

    if operation == "add":
        result = add(a, b)
    elif operation == "subtract":
        result = subtract(a, b)
    elif operation == "multiply":
        result = multiply(a, b)
    elif operation == "divide":
        result = divide(a, b)
    else:
        raise HTTPException(status_code=422, detail="Unsupported operation")

    return {"result": result}
```

**Example input / output:**

```

Example request:

HTTP
Copy
POST /calculate/ HTTP/1.1
Host: localhost:8000
Content-Type: application/json

{"operation": "add", "a": 5, "b": 3}

Response:

JSON
Copy
{
    "result": 8
}
```

**Data:**

```json
{
    "operation": "add",
    "a": 5,
    "b": 3
}
```

**Flow:**

1. yes
2. no
3. Receive Request
4. Is Operation Valid?
5. Perform Calculation
6. Return Result
7. Return Error

