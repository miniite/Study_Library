---
type: "practice"
difficulty: "Basic"
---

# How would you document your FastAPI GenAI endpoints with example prompts so consumers know what to send?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ File Uploads and Inference](<../../../../TOOLS/Python/FastAPI/File_Uploads_and_Inference.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To document FastAPI GenAI endpoints effectively, we need to ensure that users understand how to construct and send requests appropriately. This involves providing clear documentation of the endpoint’s purpose, input parameters, and example prompts. Here's a step-by-step approach to achieve this using FastAPI.

FastAPI is a modern web framework for building APIs in Python 3.10+ with minimal code. It supports automatic generation of interactive API documentation (e.g., Swagger UI or ReDoc) which can be enhanced by adding custom documentation blocks and example prompts. Let's break down the process:

#### Triage

The question asks about documenting FastAPI endpoints for GenAI, specifically focusing on providing example prompts to help consumers know what input data to send. The interviewer is probing your understanding of RESTful API design principles, documentation best practices, and how to use FastAPI’s built-in tools effectively.

#### Assumptions
You are familiar with the basics of using FastAPI.
You understand that effective documentation helps improve user experience and reduces support queries.
The GenAI model requires specific inputs, such as text prompts or structured data.

#### Solution

Define Endpoint Purpose: Clearly state what each endpoint does in its description. For example:

MARKDOWN
# /generate
This endpoint generates a response based on the input prompt using a GenAI model.

Document Input Parameters: Detail any required or optional parameters, their data types, and possible values. Use the Body annotation to specify schema validation if necessary.

Provide Example Prompts: Show sample inputs that would elicit desired responses from the GenAI model. This can be done using the @pydantic.Field decorator for clearer documentation:

PYTHON
from fastapi import FastAPI, Body
from pydantic import BaseModel

app = FastAPI()

class PromptRequest(BaseModel):
    prompt: str  # Example: "What is the capital of France?"

@app.post("/generate")
def generate_response(prompt_request: PromptRequest) -> dict:
    """Generate a response based on the input prompt."""
    response = process_prompt(prompt_request.prompt)
    return {"response": response}

Automated Documentation: Use FastAPI’s built-in auto-generated documentation tools like Swagger UI or ReDoc to provide interactive and user-friendly documentation.

Custom Documentation Blocks: Enhance the generated documentation with custom markdown blocks for more detailed explanations:

MARKDOWN
# /generate

This endpoint generates a response based on the input prompt using a GenAI model.

## Example Request
POST `/generate`

{ "prompt": "What is the capital of France?" }

CODE

## Example Response

{ "response": "Paris" }

#### Closing Summary
+ Common Mistakes/Follow-Ups

By following these steps, you ensure that your FastAPI GenAI endpoints are well-documented and easy to understand. Common mistakes include neglecting clear input descriptions or providing insufficient example prompts. Always test your documentation with real examples to catch any issues early.

For follow-ups, the interviewer might ask about handling complex data structures, error handling in responses, or integrating third-party documentation tools.

**Code:**

```python
from fastapi import FastAPI, Body
from pydantic import BaseModel

app = FastAPI()

class PromptRequest(BaseModel):
    prompt: str  # Example: "What is the capital of France?"

@app.post("/generate")
def generate_response(prompt_request: PromptRequest) -> dict:
    """Generate a response based on the input prompt."""
    response = process_prompt(prompt_request.prompt)
    return {"response": response}
```

**Example input / output:**

```

Example Request: POST /generate

Body:

CODE
Copy
{
    "prompt": "What is the capital of France?"
}

Response:

CODE
Copy
{
    "response": "Paris"
}
```

**Data:**

```json
{
    "prompt": "What is the capital of France?"
}
```

