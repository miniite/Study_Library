---
type: "practice"
difficulty: "Intermediate"
---

# How would you design a consistent error response body for your AI API so the frontend can distinguish a rate limit from an invalid prompt from a provider outage?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Handling 429 Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Handling_429_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a consistent and informative error response body for our AI API, we need to ensure that the frontend can easily distinguish between different types of errors such as rate limits, invalid prompts, and provider outages. The key is to use clear, structured JSON responses with standardized fields that convey both the type of error and its context.

The question is really probing how you handle error cases in a REST API, ensuring robustness and user-friendliness. We will make assumptions about the types of errors we want to cover and then build our solution step-by-step from first principles.

First, let's assume that an API response should include at least these fields:

status: The HTTP status code (e.g., 429 for rate limits).
message: A human-readable description of the error.
code: A machine-readable unique identifier for the error type (e.g., "RATE_LIMIT_EXCEEDED", "INVALID_PROMPT", etc.).
details: Additional context or details about the error.
Step-by-Step Solution

Define Error Codes and Messages:

We'll define a set of standard error codes and corresponding messages.
For example:
RATE_LIMIT_EXCEEDED with message "Rate limit exceeded, please try again later."
INVALID_PROMPT with message "Invalid input provided."

Structure the Error Response JSON:

Use a consistent structure for all error responses.

#### Example
of an error response body:
JSON
{
  "status": 429,
  "message": "Rate limit exceeded, please try again later.",
  "code": "RATE_LIMIT_EXCEEDED",
  "details": "Exceeded maximum requests per minute."
}

Handling Different Error Types:

For rate limits: Use HTTP 429 with a specific error code.
For invalid prompts: Use HTTP 400 with an appropriate error code.
For provider outages: Use HTTP 503 or 500 with the appropriate error code.

#### Example
Implementation
PYTHON
def handle_rate_limit_exceeded(error_details):
    return {
        "status": 429,
        "message": "Rate limit exceeded, please try again later.",
        "code": "RATE_LIMIT_EXCEEDED",
        "details": f"Exceeded maximum requests per minute. {error_details}"
    }

def handle_invalid_prompt(error_details):
    return {
        "status": 400,
        "message": "Invalid input provided.",
        "code": "INVALID_PROMPT",
        "details": error_details
    }

def handle_provider_outage():
    return {
        "status": 503,
        "message": "Provider service is currently unavailable.",
        "code": "PROVIDER_OUTAGE",
        "details": "Please try again in a few minutes."
    }

#### Common Mistakes
and Follow-Ups

Mistake: Not providing enough detail or clarity in error messages.

#### Follow-Up
Always include contextual details that can help the user understand what went wrong and how to fix it.

Mistake: Using generic HTTP status codes without specific error codes.

#### Follow-Up
Define a set of standard error codes and map them to specific status codes for clarity.

Mistake: Overcomplicating the response structure.

#### Follow-Up
Keep the response simple but informative, focusing on the essential fields (status, message, code, details).

**Code:**

```python
def handle_rate_limit_exceeded(error_details):
    return {
        "status": 429,
        "message": "Rate limit exceeded, please try again later.",
        "code": "RATE_LIMIT_EXCEEDED",
        "details": f"Exceeded maximum requests per minute. {error_details}"
    }

def handle_invalid_prompt(error_details):
    return {
        "status": 400,
        "message": "Invalid input provided.",
        "code": "INVALID_PROMPT",
        "details": error_details
    }

def handle_provider_outage():
    return {
        "status": 503,
        "message": "Provider service is currently unavailable.",
        "code": "PROVIDER_OUTAGE",
        "details": "Please try again in a few minutes."
    }
```

