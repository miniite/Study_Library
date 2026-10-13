---
type: "practice"
difficulty: "Intermediate"
---

# How would you validate and fail fast at startup if a required environment variable is missing in a FastAPI service?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Timeouts and Resource Protection](<../../../../TOOLS/Python/FastAPI/Timeouts_and_Resource_Protection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a FastAPI service fails fast during startup due to missing required environment variables, we can implement a custom on_event handler that checks for these variables and raises an exception if any are missing. This approach ensures that the application crashes early with clear error messages, making it easier to identify configuration issues.

#### Triage
What's Being Probed

The interviewer is probing your understanding of FastAPI's lifecycle events, environment variable handling, and error management. Key assumptions include:

You have a working knowledge of FastAPI.
You understand how environment variables are used in application configurations.
You can implement custom startup logic to catch critical errors.

#### Solution
First Principles
Identify Required Environment Variables: List all required environment variables for your service.
Custom on_event Handler: Use the @app.on_event("startup") decorator to define a handler that checks these variables during application startup.
Error Handling: Raise an exception if any required variable is missing, providing clear error messages.

Here’s how you can implement this in FastAPI:

PYTHON
from fastapi import FastAPI, HTTPException
import os

app = FastAPI()

# Define the required environment variables
REQUIRED_ENV_VARS = ["DB_HOST", "DB_PORT", "SECRET_KEY"]

@app.on_event("startup")
async def startup():
    # Check for each required environment variable
    for var in REQUIRED_ENV_VARS:
        if not os.getenv(var):
            raise HTTPException(
                status_code=500,
                detail=f"Required environment variable {var} is missing."
            )

# Example route to ensure the service is up and running after startup checks
@app.get("/")
async def read_root():
    return {"message": "Service is operational."}

#### Closing Summary
+ Common Mistakes/Follow-Ups

This approach ensures that your FastAPI service fails gracefully during startup if any required environment variables are missing. By using an HTTPException with a descriptive message, you provide clear diagnostics for developers deploying or troubleshooting the application.

#### Common Mistakes

Not Checking All Required Variables: Ensure all critical variables are included in the check.
Using a Generic Exception: Raising a generic exception can make debugging harder; providing specific error messages helps.
Ignoring Non-Startup Errors: This solution only handles missing environment variables during startup. Consider additional validation logic for runtime errors as well.

#### Follow-Ups

How would you handle different types of required configurations, such as database connection strings or API keys?
Can you discuss the trade-offs between failing fast and allowing some flexibility in configuration management?

**Code:**

```python
from fastapi import FastAPI, HTTPException
import os

app = FastAPI()

REQUIRED_ENV_VARS = ["DB_HOST", "DB_PORT", "SECRET_KEY"]

@app.on_event("startup")
async def startup():
    for var in REQUIRED_ENV_VARS:
        if not os.getenv(var):
            raise HTTPException(
                status_code=500,
                detail=f"Required environment variable {var} is missing."
            )

@app.get("/")
async def read_root():
    return {"message": "Service is operational."}

N/A
```

