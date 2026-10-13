---
type: "practice"
difficulty: "Deep"
---

# How do you guard a FastAPI inference endpoint against adversarial oversized or malformed feature payloads that could cause NumPy or the model to allocate huge memory?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Time-series Prediction APIs](<../../../../TOOLS/Python/FastAPI/Time-series_Prediction_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To guard a FastAPI inference endpoint from adversarial oversized or malformed feature payloads, we need to ensure robust input validation and handling of potential errors, especially when dealing with large arrays that could cause NumPy or our model to allocate excessive memory. The core idea is to implement checks at multiple levels—input size, data type integrity, and finally, memory management—to prevent the endpoint from crashing due to resource exhaustion.

First, we need to understand that FastAPI allows us to define request schemas to validate input structures, but for handling large payloads or potential NumPy issues, we must go beyond basic schema validation. Here’s a step-by-step approach:

Schema Validation: Define your input data structure using Pydantic models.
Payload Size Limiting: Use FastAPI’s built-in max_body_size to limit the maximum size of incoming requests.
Data Type Integrity Checks: Validate that the incoming payload conforms to expected data types and shapes, leveraging NumPy’s capabilities.
Memory Management: Implement custom error handling and memory monitoring.

Now let's break this down from first principles:

#### Triaging the Question

The interviewer is probing your understanding of:

How FastAPI works with large payloads.
The risks associated with handling oversized or malformed data in a production environment, particularly with NumPy-based models.
Best practices for input validation and memory management.

#### Assumptions
You are familiar with FastAPI, Python, Pydantic, and NumPy.
Your model processes numerical data as NumPy arrays.
The inference endpoint is exposed to the internet or a public API.
Step-by-Step Solution
1. Schema Validation Using Pydantic Models

Define your input schema using Pydantic models for easy validation:

PYTHON
from pydantic import BaseModel, conlist
import numpy as np

class InputData(BaseModel):
    features: conlist(float, min_items=10, max_items=1000)

This ensures that the incoming data is of the correct type and shape.

2. Limiting Payload Size with FastAPI

Use max_body_size to limit the maximum size of incoming requests:

PYTHON
from fastapi import FastAPI

app = FastAPI(max_body_size=10 * 1024 * 1024)  # 10MB max body size
3. Data Type and Shape Validation

Perform additional validation within your endpoint handler to ensure the data is of the correct type and shape before processing with NumPy:

PYTHON
@app.post("/inference")
async def inference(input_data: InputData):
    try:
        # Convert Pydantic model to NumPy array
        features = np.array(input_data.features)

        # Additional validation (e.g., check for NaN, Inf values)
        if np.isnan(features).any() or np.isinf(features).any():
            return {"error": "Input contains invalid numerical values"}

        # Model inference logic here

    except ValueError as e:
        return {"error": f"Invalid input: {str(e)}"}

    except MemoryError:
        return {"error": "Request payload too large. Please reduce the size of your data."}

    else:
        return {"result": model_inference(features)}
4. Custom Error Handling and Memory Management

Implement custom error handling to catch memory-related issues:

PYTHON
@app.exception_handler(MemoryError)
async def memory_error_handler(request, exc):
    return JSONResponse(status_code=500, content={"error": "Memory allocation failed. Please reduce the size of your data."})

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, to guard against adversarial oversized or malformed feature payloads in a FastAPI inference endpoint:

Use Pydantic models for schema validation.
Limit payload sizes with max_body_size.
Perform additional type and shape validations.
Implement custom error handling and memory monitoring.

#### Common mistakes
include not validating the input data thoroughly or relying solely on default behavior, which can leave your application vulnerable to large or malformed payloads. Always consider edge cases where users might try to exploit potential weaknesses in the system.

For a deeper discussion, you could explore other techniques such as using rate limiting, implementing more sophisticated memory management strategies, and considering asynchronous processing for extremely large datasets.

**Code:**

```python
from fastapi import FastAPI, Request
from pydantic import BaseModel, conlist
import numpy as np

app = FastAPI(max_body_size=10 * 1024 * 1024)  # 10MB max body size

class InputData(BaseModel):
    features: conlist(float, min_items=10, max_items=1000)

@app.post("/inference")
async def inference(input_data: InputData):
    try:
        features = np.array(input_data.features)

        if np.isnan(features).any() or np.isinf(features).any():
            return {"error": "Input contains invalid numerical values"}

        # Model inference logic here
        result = model_inference(features)
        return {"result": result}

    except ValueError as e:
        return {"error": f"Invalid input: {str(e)}"}

    except MemoryError:
        return {"error": "Request payload too large. Please reduce the size of your data."}

@app.exception_handler(MemoryError)
async def memory_error_handler(request, exc):
    return JSONResponse(status_code=500, content={"error": "Memory allocation failed. Please reduce the size of your data."})
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "features": [1.2, 3.4, 5.6, ...]  # 1000 elements
}

Output:

JSON
Copy
{
  "result": "Inference result"
}
```

**Flow:**

1. Valid
2. Within Limit
3. Passes
4. Invalid
5. Exceeds Limit
6. Fails
7. Receive Request
8. Validate Schema
9. Check Payload Size
10. Validate Data Type and Shape
11. Inference
12. Return Inference Result
13. Return Error: Invalid Input
14. Return Error: Request PayloadToo Large
15. Handle MemoryError: ReturnError Message

