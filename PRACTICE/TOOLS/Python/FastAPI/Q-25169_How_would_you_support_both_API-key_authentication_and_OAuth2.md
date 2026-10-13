---
type: "practice"
difficulty: "Deep"
---

# How would you support both API-key authentication and OAuth2 bearer tokens on the same FastAPI route?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ API Authentication](<../../../../TOOLS/Python/FastAPI/API_Authentication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To support both API-key authentication and OAuth2 bearer tokens on the same FastAPI route, we need to implement a flexible authentication system that can handle multiple authentication methods without conflict. This solution will ensure that users can authenticate via an API key or OAuth2 token seamlessly.

Firstly, let's understand what these authentication methods entail:

API Key Authentication: A simple method where a unique string is passed in the Authorization header.
OAuth2 Bearer Tokens: More complex and widely used, requiring a token typically obtained through the OAuth2 protocol, which can be passed either via the Authorization header or as a query parameter.

FastAPI provides powerful tools to handle these scenarios, and we'll leverage its built-in features and custom middleware for this purpose. We will create separate authentication functions for both methods and combine them using FastAPI's capabilities.

Here is a step-by-step approach:

Define Authentication Functions: Write functions to validate each type of token.
Implement Middleware: Create custom middleware to handle the decision logic between the two types of tokens.
Combine in Route Handler: Use the middleware within your route handlers to ensure both methods are supported.
Step-by-Step Implementation
1. Define Authentication Functions

We'll define functions that check if a token is valid for each authentication method.

PYTHON
from fastapi import HTTPException, Request, Header
from starlette.status import HTTP_403_FORBIDDEN
import re

async def api_key_auth(token: str = Header(None)):
    """Check API key in the header."""
    # Replace with actual validation logic
    if token != "your_api_key":
        raise HTTPException(status_code=HTTP_403_FORBIDDEN, detail="Invalid or missing API Key")

async def oauth2_bearer(token: str):
    """Check OAuth2 Bearer token."""
    try:
        # Validate the token using your OAuth2 provider's logic
        if not validate_oauth_token(token):  # Placeholder function
            raise HTTPException(status_code=HTTP_403_FORBIDDEN, detail="Invalid or missing OAuth2 Token")
    except Exception as e:
        raise HTTPException(status_code=HTTP_403_FORBIDDEN, detail=str(e))
2. Implement Middleware

Create custom middleware that decides which authentication method to use based on the Authorization header.

PYTHON
from fastapi import Request, HTTPException

async def oauth_api_middleware(request: Request, call_next):
    authorization_header = request.headers.get("authorization")

    if authorization_header:
        # Check for API key in the header
        match = re.match(r"Bearer\s+(\w+)", authorization_header)
        if match:
            token = match.group(1)
            try:
                await oauth2_bearer(token)
                return await call_next(request)
            except HTTPException as e:
                pass  # Fall back to API key

        # Check for API key in the header
        api_key_match = re.match(r"api-key\s+(\w+)", authorization_header, re.IGNORECASE)
        if api_key_match:
            token = api_key_match.group(1)
            try:
                await api_key_auth(token)
                return await call_next(request)
            except HTTPException as e:
                pass  # Return 403 or continue to next middleware

    raise HTTPException(status_code=HTTP_403_FORBIDDEN, detail="Invalid or missing Authentication")
3. Combine in Route Handler

Use the middleware within your route handler.

PYTHON
from fastapi import APIRouter

router = APIRouter()

@router.get("/protected-endpoint")
async def protected_endpoint():
    return {"message": "This is a protected endpoint"}
Full Example with Middleware and Routes
PYTHON
from fastapi import FastAPI, Request, HTTPException
import re
from fastapi.middleware.base import BaseHTTPMiddleware

app = FastAPI()

# Define authentication functions
async def api_key_auth(token: str = Header(None)):
    # Replace with actual validation logic
    if token != "your_api_key":
        raise HTTPException(status_code=403, detail="Invalid or missing API Key")

async def oauth2_bearer(token: str):
    try:
        # Validate the token using your OAuth2 provider's logic
        if not validate_oauth_token(token):  # Placeholder function
            raise HTTPException(status_code=403, detail="Invalid or missing OAuth2 Token")
    except Exception as e:
        raise HTTPException(status_code=403, detail=str(e))

# Implement middleware
class OAuthAPIMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        authorization_header = request.headers.get("authorization")

        if authorization_header:
            # Check for API key in the header
            match = re.match(r"Bearer\s+(\w+)", authorization_header)
            if match:
                token = match.group(1)
                try:
                    await oauth2_bearer(token)
                    return await call_next(request)
                except HTTPException as e:
                    pass  # Fall back to API key

            # Check for API key in the header
            api_key_match = re.match(r"api-key\s+(\w+)", authorization_header, re.IGNORECASE)
            if api_key_match:
                token = api_key_match.group(1)
                try:
                    await api_key_auth(token)
                    return await call_next(request)
                except HTTPException as e:
                    pass  # Return 403 or continue to next middleware

        raise HTTPException(status_code=403, detail="Invalid or missing Authentication")

app.middleware("http")(OAuthAPIMiddleware())

# Define route
@router.get("/protected-endpoint")
async def protected_endpoint():
    return {"message": "This is a protected endpoint"}

#### Closing Summary
and Common Mistakes

By following this approach, we ensure that both API-key and OAuth2 bearer tokens can be used interchangeably without conflicts. This solution leverages FastAPI's built-in middleware capabilities to seamlessly handle multiple authentication methods.

#### Common Mistakes

Not validating the token properly.
Failing to catch exceptions properly in middleware functions.
Overlapping logic that may cause unnecessary 403 errors or incorrect handling of tokens.

