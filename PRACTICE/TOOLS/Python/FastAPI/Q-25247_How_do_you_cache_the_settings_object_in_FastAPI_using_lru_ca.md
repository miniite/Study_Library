---
type: "practice"
difficulty: "Intermediate"
---

# How do you cache the settings object in FastAPI using lru_cache so environment parsing happens only once?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Caching Predictions](<../../../../TOOLS/Python/FastAPI/Caching_Predictions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To cache the settings object in a FastAPI application, we can use lru_cache to ensure that environment parsing happens only once and subsequent requests reuse the cached settings. This approach is efficient and leverages Python's built-in caching mechanism.

#### Triage

The question is about optimizing the initialization of the settings object by caching it using lru_cache. The interviewer might be probing your understanding of how lru_cache works, the trade-offs between caching and re-parsing environment variables, and best practices for setting up such a cache in FastAPI.

#### First-Principles Solution
Understanding lru_cache: lru_cache is a decorator provided by Python's functools module that caches the results of a function call to avoid redundant computations.
Environment Parsing: In FastAPI, environment variables are often parsed using .env files or other configuration methods like os.environ. We need to ensure this parsing happens only once during the application startup.
Implementing Cache:
Define a function that parses and returns the settings object.
Decorate this function with lru_cache.
Initialization: Ensure that the cached settings are available throughout the application lifecycle.
Step-by-Step Solution:

Install Required Libraries:

BASH
pip install python-dotenv

Define Settings Class:

PYTHON
from fastapi import FastAPI
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    API_KEY: str
    DEBUG_MODE: bool

    model_config = SettingsConfigDict(env_file='.env')

Parse Environment Variables Using lru_cache:

PYTHON
from functools import lru_cache
from fastapi import FastAPI, HTTPException

app = FastAPI()

@lru_cache()
def get_settings() -> Settings:
    return Settings()

Use Cached Settings in Routes:

PYTHON
@app.get("/settings")
async def read_root(settings=get_settings):
    if not settings:
        raise HTTPException(status_code=500, detail="Settings are not initialized.")
    return {"api_key": settings.API_KEY, "debug_mode": settings.DEBUG_MODE}

#### Closing Summary
and Common Mistakes/Follow-Ups:
Summary: By using lru_cache, we ensure that the environment variables are parsed only once during startup. This reduces unnecessary re-parsing on every request.

#### Common Mistakes
Not properly handling exceptions if the settings object is not initialized.
Forgetting to import and use the cached function (get_settings) in routes.
Questions for Follow-Up:
How would you handle different environments (development, staging, production)?
What are the potential issues with caching environment variables?
How can you ensure that your application remains flexible if configuration changes frequently?

**Code:**

```python
from fastapi import FastAPI
from pydantic_settings import BaseSettings, SettingsConfigDict
from functools import lru_cache

class Settings(BaseSettings):
    API_KEY: str
    DEBUG_MODE: bool

    model_config = SettingsConfigDict(env_file='.env')

@lru_cache()
def get_settings() -> Settings:
    return Settings()

app = FastAPI()

@app.get("/settings")
async def read_root(settings=get_settings):
    if not settings:
        raise HTTPException(status_code=500, detail="Settings are not initialized.")
    return {"api_key": settings.API_KEY, "debug_mode": settings.DEBUG_MODE}
```

**Example input / output:**

```text
GET /settings

Response:
{
  "api_key": "my_api_key",
  "debug_mode": false
}
```

**Data:**

```
ENV
Copy
API_KEY=my_api_key
DEBUG_MODE=false
```

