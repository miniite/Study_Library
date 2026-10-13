---
type: "practice"
difficulty: "Intermediate"
---

# How would you containerize a FastAPI application with Docker for deployment?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Packaging Models in Containers](<../../../../TOOLS/Python/FastAPI/Packaging_Models_in_Containers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To containerize a FastAPI application using Docker, we need to ensure that our application runs reliably and consistently across different environments. This involves setting up a Dockerfile, configuring environment variables, handling dependencies, and exposing ports. The goal is to create a lightweight, self-contained executable image that can be easily deployed on any system supporting Docker.

The interviewer is probing your understanding of the Docker containerization process, including best practices for structuring a Dockerfile, managing dependencies, and ensuring environmental consistency. Key assumptions include familiarity with Docker basics such as images, containers, and volumes.

We will walk through creating a simple Dockerfile that sets up our FastAPI application environment from scratch. We'll also discuss how to expose the necessary ports and set environment variables if required. Let's begin by understanding each step in detail:

Setup Directory Structure:

CODE
/project
    ├── app/
    │   └── main.py
    └── Dockerfile

Create a Simple FastAPI Application (main.py):

PYTHON
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello World"}

Dockerfile: We will create a Dockerfile that sets up the environment and installs all necessary dependencies.

DOCKERFILE
# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "main.py"]

Create a requirements.txt File: Ensure you have a requirements.txt file that lists all dependencies required by your application, for example:

CODE
fastapi==0.75.1
uvicorn==0.18.2

Build the Docker Image: Run the following command in the root of your project directory to build the Docker image:

BASH
docker build -t my-fastapi-app .

Run the Docker Container: Once the image is built, you can run it using:

BASH
docker run -d --name fastapi-container -p 8000:80 my-fastapi-app

Verify the Application: You should be able to access your FastAPI application by navigating to http://localhost:8000 in your browser.

By following these steps, we have containerized a FastAPI application with Docker, ensuring it is easy to deploy and run consistently across different environments.

**Code:**

```
Dockerfile for FastAPI app
Use an official Python runtime as a parent image

FROM python:3.9-slim

Set the working directory in the container

WORKDIR /app

Copy the current directory contents into the container at /app

COPY . /app

Install any needed packages specified in requirements.txt

RUN pip install --no-cache-dir -r requirements.txt

Make port 80 available to the world outside this container

EXPOSE 80

Define environment variable

ENV NAME World

Run app.py when the container launches

CMD ["python", "main.py"]

CODE
Copy

```bash
# Sample command to build and run the Docker image
docker build -t my-fastapi-app .
docker run -d --name fastapi-container -p 8000:80 my-fastapi-app
```

