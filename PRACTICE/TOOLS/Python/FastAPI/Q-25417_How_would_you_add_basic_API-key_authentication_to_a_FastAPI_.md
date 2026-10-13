---
type: "practice"
difficulty: "Basic"
---

# How would you add basic API-key authentication to a FastAPI model-serving endpoint so only authorized clients can call it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ API Authentication](<../../../../TOOLS/Python/FastAPI/API_Authentication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To secure a FastAPI model-serving endpoint using API-key authentication, we need to ensure that only clients with valid API keys are able to make requests. This involves creating an authentication mechanism within our FastAPI application. Here's a step-by-step guide on how to achieve this:

#### Overview
We will use FastAPI's built-in dependency injection and security features to add API-key authentication. The key idea is to define a custom dependency that checks the incoming request for a valid API key, and if it fails, returns an unauthorized response.

#### Triage

What the question is asking: Implement a method to authenticate clients by validating their API keys before allowing access to the model-serving endpoint.
What the interviewer might be probing: Your understanding of FastAPI dependencies, handling HTTP requests, and implementing security features.

#### Assumptions
We assume you have basic knowledge of Python and FastAPI.

#### First-Principles Solution

Define a custom dependency to check API keys.
Store the valid API keys in a secure manner (e.g., environment variables or a database).
Use this dependency in your route definitions to enforce authentication.

#### Implementation
Steps:

First, install FastAPI if you haven't already: pip install fastapi.
Define a custom dependency that checks the incoming request for an API key.
PYTHON
from typing import Optional

from fastapi import Depends, HTTPException, status
from fastapi.security.api_key import APIKeyHeader
from starlette.requests import Request

api_key_header = APIKeyHeader(name="X-API-KEY", auto_error=True)

def get_api_key(api_key_header: str = Depends(api_key_header)) -> Optional[str]:
    # Replace 'VALID_API_KEYS' with a set of valid keys or implement your own logic to fetch them.
    VALID_API_KEYS = {"your_valid_api_key"}

    if api_key_header in VALID_API_KEYS:
        return api_key_header
    else:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid API key",
            headers={"WWW-Authenticate": "Bearer"},
        )

Using the Custom Dependency:

Apply this custom dependency to your endpoint definitions.
PYTHON
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.get("/model/predict")
def predict(api_key: str = Depends(get_api_key)):
    # Here you would have code to handle the prediction logic using a model.
    return {"message": "Prediction successful"}

#### Closing Summary
+ Common Mistakes/Follow-Ups:

This approach ensures that only clients with valid API keys can access your endpoint. The key points are defining a custom dependency and applying it correctly to enforce authentication.

#### Common mistakes
include not handling invalid API keys properly (e.g., returning an error message), or hardcoding the API keys in the code, which poses security risks.

#### Follow-ups
might involve discussing alternative methods of securing APIs (like OAuth2) or strategies for managing multiple environment configurations with different sets of valid keys.

