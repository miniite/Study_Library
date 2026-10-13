---
type: "practice"
difficulty: "Basic"
---

# How do you set a custom HTTP status code for a FastAPI response?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Custom Responses](<../../../../TOOLS/Python/FastAPI/Custom_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with FastAPI, setting a custom HTTP status code is straightforward and can be done through the status_code parameter in your API endpoint function. This feature allows you to explicitly define what status code should accompany the response data based on the specific needs of your application. The core idea here is that you have fine-grained control over not just the payload but also the HTTP semantics, which can greatly enhance the robustness and clarity of your API interactions.

To start, let's understand what exactly the question is asking: FastAPI itself doesn't require any special setup to support custom status codes; it's a natural part of how the framework works. We're essentially probing your understanding of HTTP status codes and your familiarity with using them in context with a modern web framework like FastAPI.

We'll make two assumptions:

You are already familiar with basic FastAPI usage.
The status code you want to set is predefined by the http.HTTPStatus class or an integer value corresponding to an HTTP status.

Now, let's walk through this from first principles:

PYTHON
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.get("/custom-status-code")
def custom_status_code():
    # Here we are setting a custom 401 (Unauthorized) status code.
    # You can replace `HTTP_401_UNAUTHORIZED` with any other valid status code.
    raise HTTPException(status_code=401, detail="Custom message for unauthorized access.")

In the example above, the function custom_status_code uses an HTTPException to return a 401 Unauthorized status. This is just one way; you can also directly set the response's status code without raising an exception:

PYTHON
from fastapi import FastAPI, Response

app = FastAPI()

@app.get("/custom-status-code")
def custom_status_code(response: Response):
    # Directly setting a 401 Unauthorized status.
    response.status_code = 401
    return {"detail": "Custom message for unauthorized access."}

Both approaches achieve the same goal, but using HTTPException can be more idiomatic in FastAPI and easier to handle within your application.

#### Closing summary
Setting a custom HTTP status code in FastAPI is achieved through either an HTTPException or directly setting the response's status code. Understanding when and why you would use each approach is key to building robust APIs.

#### Common mistakes/follow-ups

Using incorrect HTTP status codes that don't align with the semantics of your API.
Forgetting to include a meaningful message in exceptions, which can be useful for debugging or client-side handling.
Misusing HTTPException where direct setting might suffice and vice versa.

**Code:**

```

from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.get("/custom-status-code") def custom_status_code(): raise HTTPException(status_code=401, detail="Custom message for unauthorized access.")

CODE
Copy

```python
from fastapi import FastAPI, Response

app = FastAPI()

@app.get("/custom-status-code")
def custom_status_code(response: Response):
    response.status_code = 401
    return {"detail": "Custom message for unauthorized access."}
```

