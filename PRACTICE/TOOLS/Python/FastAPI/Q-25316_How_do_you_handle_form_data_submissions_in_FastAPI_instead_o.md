---
type: "practice"
difficulty: "Intermediate"
---

# How do you handle form data submissions in FastAPI instead of JSON bodies?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Custom Responses](<../../../../TOOLS/Python/FastAPI/Custom_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle form data submissions in FastAPI, we need to understand how FastAPI processes different types of requests and how to properly configure it to work with form data. This is crucial because by default, FastAPI expects JSON bodies but can be configured to accept other formats like form data.

#### Triage

The question is asking for a method to handle form data submissions in FastAPI instead of the usual JSON body approach. The interviewer might be probing your understanding of FastAPI's request handling mechanisms and your ability to customize them according to specific requirements. Key assumptions include familiarity with HTTP methods, FastAPI's routing configuration, and form data processing.

#### First-Principles Solution
Understanding Request Types: In FastAPI, a typical request body is expected to be in JSON format when using the POST method for form data submissions.
Using Pydantic Models: We can define models that map directly to the form data structure and use them as input parameters for our API endpoints.
Handling Form Data Explicitly: Use the Form() annotation from FastAPI to explicitly handle form data.

#### Example
Code

Let's consider a scenario where we have an endpoint to accept user login credentials in a form:

PYTHON
from fastapi import FastAPI, Form
from pydantic import BaseModel

app = FastAPI()

# Define a Pydantic model for the expected form data
class LoginForm(BaseModel):
    username: str
    password: str

@app.post("/login")
async def login(username: str = Form(...), password: str = Form(...)):
    # Process the form data here
    print(f"Received login request with username: {username} and password: {password}")
    return {"message": "Login successful"}

#### Closing Summary
+ Common Mistakes/Follow-Ups

In this example, we used FastAPI's Form() annotation to handle form data explicitly. By default, FastAPI will treat the incoming request body as JSON if no specific handling is done for non-JSON content types. To handle form data correctly, we need to ensure that our route definitions and form fields are appropriately annotated.

#### Common mistakes
include:

Forgetting to use Form(): This can lead to incorrect parsing of form data.
Not using Pydantic models properly: Mismatching the structure of the form data with the model can result in unexpected behavior or errors.
Misconfiguring the request body type: Ensuring that the content-type header is set correctly (e.g., application/x-www-form-urlencoded).

#### Follow-ups
might include discussing additional configuration options, handling file uploads, and ensuring compatibility with different HTTP clients.

