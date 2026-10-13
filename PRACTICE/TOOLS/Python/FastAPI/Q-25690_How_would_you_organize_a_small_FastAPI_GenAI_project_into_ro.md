---
type: "practice"
difficulty: "Basic"
---

# How would you organize a small FastAPI GenAI project into routers, schemas, and an llm service module?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ LLM Endpoints in FastAPI](<../../../../TOOLS/Python/FastAPI/LLM_Endpoints_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this FastAPI GenAI project, we will organize the codebase by creating clear, modular components such as routers, schemas, and a dedicated LLM (Language Model) service. This approach not only makes the application more maintainable but also easier to scale and test.

To break down the question: The interviewer is probing how you structure an AI project using FastAPI, focusing on best practices for organization and modularity. We will assume that this GenAI project involves some form of language processing or text generation tasks using a pre-trained model like GPT-3 or another LLM.

Let’s start by outlining the key components we need to create:

Routers: These handle API requests and route them to the appropriate functions.
Schemas: Define data models for input and output.
LLM Service Module: Manages interactions with the language model, including setup, configuration, and handling of responses.

Now let’s go through each component step-by-step:

Routers

Routers in FastAPI are used to define routes that handle HTTP requests. We will create separate routers for different functionalities such as text generation, sentiment analysis, etc.

PYTHON
# app/routers/text_generation.py
from fastapi import APIRouter

router = APIRouter()

@router.post("/generate-text")
async def generate_text(prompt: str):
    # Call LLM service and return generated text
    pass
Schemas

Schemas define the structure of input data, making it easier to validate user inputs. We will create Pydantic models for our schemas.

PYTHON
# app/schemas/text_generation.py
from pydantic import BaseModel

class TextGenerationRequest(BaseModel):
    prompt: str

class TextGenerationResponse(BaseModel):
    generated_text: str
LLM Service Module

This module will handle interactions with the language model, ensuring that we manage any potential complexities such as rate limiting or error handling.

PYTHON
# app/services/llm_service.py
import openai  # Assuming OpenAI API for demonstration

class LLMServices:
    def __init__(self):
        self.api_key = "YOUR_API_KEY"

    async def generate_text(self, prompt: str) -> str:
        response = openai.Completion.create(
            engine="davinci",
            prompt=prompt,
            max_tokens=50
        )
        return response.choices[0].text.strip()
Main Application

Finally, we will integrate these components into our main application file.

PYTHON
# app/main.py
from fastapi import FastAPI
from routers.text_generation import router as text_generation_router
from services.llm_service import LLMServices

app = FastAPI()

@app.on_event("startup")
async def startup_event():
    # Initialize LLM service on application startup
    global llm_services
    llm_services = LLMServices()

app.include_router(text_generation_router, prefix="/api/v1")

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)

#### Closing Summary
and Common Mistakes/Follow-Ups

This structure ensures that our application is modular and easy to extend. By separating concerns into routers, schemas, and a service layer, we can maintain a clean architecture.

#### Common mistakes
to avoid:

Not using Pydantic models for input validation.
Failing to initialize services at the correct stage (e.g., startup event).
Mixing business logic with routing in the same module.

In follow-ups, the interviewer might ask about handling errors, testing strategies, or scaling considerations. Make sure to address these points if they come up.

**Code:**

```python
# app/routers/text_generation.py
from fastapi import APIRouter

router = APIRouter()

@router.post("/generate-text")
async def generate_text(prompt: str):
    # Call LLM service and return generated text
    pass

# app/schemas/text_generation.py
from pydantic import BaseModel

class TextGenerationRequest(BaseModel):
    prompt: str

class TextGenerationResponse(BaseModel):
    generated_text: str

# app/services/llm_service.py
import openai  # Assuming OpenAI API for demonstration

class LLMServices:
    def __init__(self):
        self.api_key = "YOUR_API_KEY"

    async def generate_text(self, prompt: str) -> str:
        response = openai.Completion.create(
            engine="davinci",
            prompt=prompt,
            max_tokens=50
        )
        return response.choices[0].text.strip()

# app/main.py
from fastapi import FastAPI
from routers.text_generation import router as text_generation_router
from services.llm_service import LLMServices

app = FastAPI()

@app.on_event("startup")
async def startup_event():
    # Initialize LLM service on application startup
    global llm_services
    llm_services = LLMServices()

app.include_router(text_generation_router, prefix="/api/v1")

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

