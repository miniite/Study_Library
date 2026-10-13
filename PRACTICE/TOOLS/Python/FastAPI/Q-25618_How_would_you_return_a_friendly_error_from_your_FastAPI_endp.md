---
type: "practice"
difficulty: "Intermediate"
---

# How would you return a friendly error from your FastAPI endpoint when the LLM API responds with a rate-limit or 429 status?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Rate Limits and Backpressure](<../../../../TOOLS/Python/FastAPI/Rate_Limits_and_Backpressure.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle rate-limit errors returned by an LLM API in a FastAPI endpoint, we need to ensure that our response is user-friendly and informative. This involves catching the specific error from the LLM API, transforming it into a friendly message, and then returning this message with an appropriate HTTP status code.

#### Triage
What's Being Probed

The question is asking how to gracefully handle rate-limit errors in FastAPI by providing a human-readable response. The key aspects are:

Error Handling: How to catch the 429 (Too Many Requests) error from the LLM API.
Response Crafting: How to format and return a friendly message that informs the user about the issue.
HTTP Status Codes: Understanding when to use specific status codes like HTTP_429_TOO_MANY_REQUESTS.

#### Assumptions
We are using FastAPI, which is built on top of Starlette and supports modern web development practices with Python.
The LLM API returns a JSON response containing an error message or code that indicates it has been rate-limited.

#### First Principles Solution

Define the Endpoint: Create a FastAPI endpoint that makes a request to the LLM API.

Handle 429 Error: Catch the HTTP 429 status and handle it appropriately.

Return Friendly Response:

Use HTTPException from FastAPI to return a user-friendly error message.
Customize the error response with a meaningful title, detail, and additional context if needed.

#### Implementation

Here’s an example of how you could implement this in FastAPI:

PYTHON
from fastapi import FastAPI, HTTPException
import requests
from pydantic import BaseModel

app = FastAPI()

class LLMResponse(BaseModel):
    error: str  # Assuming the LLM API returns an 'error' field for rate limits.

@app.post("/call_llm_api")
async def call_llm_api():
    try:
        response = requests.get("https://api.example.com/llm", timeout=5)
        llm_response = LLMResponse(**response.json())

        # Handle 429 error
        if response.status_code == 429:
            raise HTTPException(
                status_code=response.status_code,
                detail="Rate limit exceeded. Please try again later.",
                headers={"Retry-After": str(response.headers.get('Retry-After', '60'))}
            )

        # Continue processing the response
        return llm_response

    except requests.exceptions.RequestException as e:
        raise HTTPException(
            status_code=500,
            detail="An error occurred while calling the LLM API. Please try again.",
            headers={"Retry-After": "30"}
        )

#### Closing Summary

In summary, to return a friendly error from your FastAPI endpoint when an LLM API responds with a rate-limit or 429 status, you need to catch this specific HTTP response, transform it into a user-friendly message, and return it along with the appropriate HTTP_429_TOO_MANY_REQUESTS status code. This approach ensures that users get clear feedback without breaking their experience.

#### Common Mistakes/Follow-Ups
Forgetting to Handle Specific Status Codes: Always check for specific error codes (like 429) and handle them separately.
Not Including Retry-After Header: Adding the Retry-After header can help users understand when they should retry their request.

**Code:**

```python
from fastapi import FastAPI, HTTPException
import requests
from pydantic import BaseModel

app = FastAPI()

class LLMResponse(BaseModel):
    error: str  # Assuming the LLM API returns an 'error' field for rate limits.
```

