---
type: "practice"
difficulty: "Basic"
---

# What are Swagger UI and ReDoc and how do you access them in a FastAPI app?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Custom Responses](<../../../../TOOLS/Python/FastAPI/Custom_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To start, let's quickly summarize what Swagger UI and ReDoc are and their role in API development with FastAPI. Swagger UI is a graphical user interface (GUI) that automatically generates from OpenAPI/Swagger specifications to help developers interact with APIs. ReDoc is an open-source alternative to Swagger UI, also providing this kind of interactive documentation but with its own unique features. Both tools make it easier for developers and other stakeholders to understand and use the API by offering a visual interface.

Now, let's dive into how you can access these interfaces in a FastAPI application. When building a FastAPI app, we leverage the FastAPI framework’s built-in support for OpenAPI generation, which is essential for generating both Swagger UI and ReDoc documentation.

Firstly, let’s assume you have a basic FastAPI app with some endpoints defined:

PYTHON
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def read_root():
    return {"message": "Hello World"}

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}

To access Swagger UI and ReDoc, you need to install the fastapi package if not already done (though it’s usually included in the FastAPI setup). The key step is adding a route that serves the generated documentation.

Here’s how to enable Swagger UI:

Install any necessary dependencies: Ensure your environment has python and pip.
Create or modify your main file, typically main.py, to include the following lines at the bottom of your code:
PYTHON
if __name__ == "__main__":
    import uvicorn

    # This enables Swagger UI and ReDoc by default on http://127.0.0.1:8000/docs and /redoc respectively.
    uvicorn.run(app, host="0.0.0.0", port=8000)

When you run your application using python main.py, Swagger UI will be available at http://localhost:8000/docs by default. ReDoc can be accessed via http://localhost:8000/redoc.

#### Triaging the Question

The interviewer is likely probing your understanding of FastAPI’s auto-generated documentation and how to enable it for better API exploration. They are also looking at whether you grasp the difference between Swagger UI and ReDoc, both in terms of functionality and appearance.

#### Assumptions
You understand basic FastAPI setup.
Familiarity with Python web frameworks and HTTP requests.
Reasoning from First Principles

Starting with a simple FastAPI app, we know that it inherently supports generating an OpenAPI specification. By default, this generates documentation accessible via Swagger UI at /docs. However, if you prefer ReDoc, which offers a different layout and additional features like offline support, you can switch to it by navigating to the redoc path.

#### Closing Summary
and Common Mistakes

In summary, integrating Swagger UI or ReDoc into your FastAPI application is straightforward. Ensure you run your app with a server (like Uvicorn) that serves the generated documentation on the default paths. A common mistake might be forgetting to include the if __name__ == "__main__": block for running the app from the command line, leading to no documentation being served.

