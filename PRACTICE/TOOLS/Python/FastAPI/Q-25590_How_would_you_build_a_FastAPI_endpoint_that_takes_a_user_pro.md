---
type: "practice"
difficulty: "Basic"
---

# How would you build a FastAPI endpoint that takes a user prompt and returns a completion from an LLM provider like OpenAI or Anthropic?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ LLM Endpoints in FastAPI](<../../../../TOOLS/Python/FastAPI/LLM_Endpoints_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a FastAPI endpoint that takes a user prompt and returns a completion from an LLM provider such as OpenAI or Anthropic, we need to follow these steps:

#### Overview
The task involves setting up a FastAPI server that receives HTTP requests with user prompts and sends them to an LLM provider for processing. The response is then returned to the client.

#### Triage
The question asks us to implement a FastAPI endpoint, which means understanding how to set up a web framework in Python using FastAPI, making API calls to the LLM provider, handling errors, and ensuring security.

#### Assumptions

We will use OpenAI as our example LLM provider.
The user prompt will be received via an HTTP POST request.
The response from the LLM is expected to be in JSON format.
Step-by-Step Solution:
Setup FastAPI: Install FastAPI and its dependencies using pip.
Create a Simple API Endpoint: Define the endpoint that accepts user prompts.
Integrate with LLM Provider: Use an appropriate HTTP client (like requests or httpx) to make API calls to OpenAI.
Handle Responses and Errors: Process the response from the LLM provider and return it to the client.

Here’s a detailed breakdown of each step:

Step 1: Setup FastAPI

BASH
pip install fastapi[all] uvicorn

Step 2: Create a Simple API Endpoint

We will define an endpoint that accepts a user prompt in JSON format. The response from the LLM provider is expected to be in JSON as well.

PYTHON
from fastapi import FastAPI, HTTPException
import requests

app = FastAPI()

@app.post("/prompt/")
async def get_completion(prompt: str):
    try:
        # Make API call to OpenAI
        response = requests.post(
            "https://api.openai.com/v1/engines/davinci-codex/completions",
            headers={
                "Content-Type": "application/json",
                "Authorization": f"Bearer YOUR_API_KEY"
            },
            json={"prompt": prompt, "max_tokens": 50}
        )

        if response.status_code == 200:
            return response.json()
        else:
            raise HTTPException(status_code=response.status_code, detail="Error from LLM provider")
    except Exception as e:
        # Handle exceptions and provide a generic error message
        raise HTTPException(status_code=500, detail=str(e))

Step 3: Integrate with LLM Provider

In the above code snippet, we use requests.post to send our user prompt to OpenAI’s API. The headers include the authorization key for authentication.

Step 4: Handle Responses and Errors

We check if the response status is 200 (OK). If not, an HTTPException is raised with the appropriate error message. Additionally, we catch any generic exceptions that may occur during the request.

#### Closing Summary
In summary, building a FastAPI endpoint to interact with an LLM provider like OpenAI involves setting up a web server using FastAPI, defining endpoints, making API calls, and handling responses and errors gracefully. This approach ensures that your application is robust and can communicate effectively with external services.

#### Common Mistakes/Follow-Ups
Common pitfalls include not handling exceptions properly, missing authentication headers, or misinterpreting the response format from the LLM provider. For follow-ups, one could explore using a more advanced client library like httpx for better performance, integrating rate limiting, and securing API keys.

