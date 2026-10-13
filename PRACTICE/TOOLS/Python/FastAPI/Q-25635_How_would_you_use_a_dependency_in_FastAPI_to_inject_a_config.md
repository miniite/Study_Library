---
type: "practice"
difficulty: "Intermediate"
---

# How would you use a dependency in FastAPI to inject a configured LLM client into every endpoint that needs it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ LLM Endpoints in FastAPI](<../../../../TOOLS/Python/FastAPI/LLM_Endpoints_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we want to ensure that an LLM (Language Model) client is easily accessible and configurable across multiple endpoints without duplicating code or hardcoding the configuration. FastAPI provides a flexible way to achieve this using dependencies.

Firstly, let's understand what dependency injection means in the context of FastAPI:

Dependency Injection: This involves passing objects as parameters to functions that need them instead of creating those objects within the function itself.
In our case, it will be an LLM client which needs to be passed into each endpoint that requires it.

Now let's break down the steps and write code from first principles:

Define the Dependency: We start by defining a class or factory function for our LLM client.
Create the Endpoint: Define endpoints that use this dependency.
Set Up Dependency Injection in FastAPI: Use the dependencies parameter to inject the dependency into the endpoint.

#### Triaging the Question

The interviewer is likely probing how you manage state and dependencies within an application, especially in a modular way. We need to ensure the LLM client can be easily configured and reused across multiple endpoints without cluttering each with its own instantiation logic.

#### Assumptions

The LLM client might require configuration such as API keys or endpoint URLs.
FastAPI will handle dependency injection seamlessly for us.
Step-by-Step Solution
1. Define the Dependency

We can define a factory function that creates and returns an instance of our LLM client. This function will be used by FastAPI to inject the object into endpoints:

PYTHON
from fastapi import Depends, HTTPException
import requests

class LLMClient:
    def __init__(self, api_key: str):
        self.api_key = api_key

    def query(self, prompt: str) -> dict:
        response = requests.post(
            "https://api.example.com/llm",
            json={"prompt": prompt},
            headers={"Authorization": f"Bearer {self.api_key}"},
        )
        if response.status_code != 200:
            raise HTTPException(status_code=response.status_code, detail="Failed to query LLM")
        return response.json()

def get_llm_client(api_key: str = Depends()) -> LLMClient:
    return LLMClient(api_key)
2. Create the Endpoint

Next, we define an endpoint that uses this dependency:

PYTHON
from fastapi import APIRouter, HTTPException

router = APIRouter()

@router.get("/query_llm")
def query_llm(llm_client: LLMClient = Depends(get_llm_client), prompt: str):
    try:
        response = llm_client.query(prompt)
        return {"response": response}
    except HTTPException as e:
        raise e
3. Set Up Dependency Injection in FastAPI

Finally, we set up the dependency injection in our main application file:

PYTHON
from fastapi import FastAPI

app = FastAPI()
app.include_router(router)

# Example usage of dependency injection
@app.on_event("startup")
def startup():
    # Here you can initialize and configure your LLM client with a real API key
    api_key = "your_api_key_here"
    get_llm_client(api_key)

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, we've used dependency injection in FastAPI to ensure that our LLM client is easily accessible and configurable across multiple endpoints. This approach keeps the code clean and modular.

#### Common Mistakes

Hardcoding the API key directly into the endpoint logic.
Not setting up proper error handling for the LLM client.
Failing to properly initialize the dependency in the application startup process.

#### Follow-Ups

How would you secure the API key used by your LLM client?
What if the LLM service has rate limits or requires authentication tokens beyond just the API key?
If you need to support multiple types of clients (e.g., different services or versions), how would you refactor this solution?

**Code:**

```python
from fastapi import Depends, HTTPException
import requests

class LLMClient:
    def __init__(self, api_key: str):
        self.api_key = api_key

    def query(self, prompt: str) -> dict:
        response = requests.post(
            "https://api.example.com/llm",
            json={"prompt": prompt},
            headers={"Authorization": f"Bearer {self.api_key}"},
        )
        if response.status_code != 200:
            raise HTTPException(status_code=response.status_code, detail="Failed to query LLM")
        return response.json()

def get_llm_client(api_key: str = Depends()) -> LLMClient:
    return LLMClient(api_key)

from fastapi import APIRouter, HTTPException

router = APIRouter()

@router.get("/query_llm")
def query_llm(llm_client: LLMClient = Depends(get_llm_client), prompt: str):
    try:
        response = llm_client.query(prompt)
        return {"response": response}
    except HTTPException as e:
        raise e

from fastapi import FastAPI

app = FastAPI()
app.include_router(router)

# Example usage of dependency injection
@app.on_event("startup")
def startup():
    # Here you can initialize and configure your LLM client with a real API key
    api_key = "your_api_key_here"
    get_llm_client(api_key)
```

