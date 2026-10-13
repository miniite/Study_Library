---
type: "practice"
difficulty: "Intermediate"
---

# How would you containerize a FastAPI ML model server with Docker so the model file is included in the image?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Packaging Models in Containers](<../../../../TOOLS/Python/FastAPI/Packaging_Models_in_Containers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To containerize a FastAPI ML model server using Docker, we need to package all necessary components into a single container that can be easily deployed and run anywhere. The key idea here is to ensure that our model file is included in the Docker image so that it's readily accessible by the application.

The question primarily asks for a step-by-step process to include the ML model in the FastAPI server container. We'll assume we have a trained ML model saved as a file, and a basic FastAPI server set up to serve this model via API endpoints. The core steps are:

Prepare the Dockerfile.
Copy the required files into the Docker image.
Build and run the Docker container.

Let's break down these steps in detail:

#### Triage
What the Question is Asking

The interviewer is probing your understanding of Docker, FastAPI, and how to package an application along with its dependencies into a container. They are also interested in seeing if you can identify potential pitfalls like missing files or incorrect paths within the Docker image.

#### Assumptions
We have a trained ML model saved as model.pkl.
A basic FastAPI server is already set up and can be run locally.
The model file path is known and consistent.
You are familiar with Docker basics such as creating a Dockerfile.
Step-by-Step Solution
1. Prepare the Dockerfile

Start by creating a Dockerfile in your project directory:

DOCKERFILE
# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set the working directory inside the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Expose port 80 for the API server to listen on
EXPOSE 80

# Run app.py when the container launches
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "80"]
2. Include the Model File in the Docker Image

Ensure that your FastAPI server is designed to load and use this model file during initialization. For example, if you are using a custom main.py script, make sure it loads the model at startup:

PYTHON
# app/main.py
from fastapi import FastAPI
import joblib

app = FastAPI()

model = joblib.load("model.pkl")  # Make sure this path is correct

@app.get("/")
def read_root():
    return {"Hello": "World"}

@app.get("/predict")
def predict(data):
    prediction = model.predict([data])
    return {"prediction": str(prediction[0])}
3. Build and Run the Docker Container

Once you have set up your Dockerfile and FastAPI server, build the Docker image:

BASH
docker build -t fastapi-ml-server .

Then run the container:

BASH
docker run -p 80:80 fastapi-ml-server

#### Closing Summary
+ Common Mistakes/Follow-ups

By following these steps, you ensure that your FastAPI ML model server is packaged in a Docker image with all necessary dependencies and the model file included. Potential pitfalls include incorrect paths to files or missing environment variables.

#### Common mistakes

Forgetting to update the WORKDIR path in the Dockerfile.
Not correctly specifying the entry point for running the application (e.g., using the correct command to run uvicorn).
Missing dependencies that are required by your FastAPI app but not included in the container.

#### Follow-ups
could include discussing how you would handle versioning of models, logging, or deploying this service into a Kubernetes cluster.

**Code:**

```dockerfile

FROM python:3.9-slim

WORKDIR /app

COPY . /app

RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 80

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "80"]

CODE
Copy

```python
# main.py
from fastapi import FastAPI
import joblib

app = FastAPI()

model = joblib.load("model.pkl")

@app.get("/")
def read_root():
    return {"Hello": "World"}

@app.get("/predict")
def predict(data):
    prediction = model.predict([data])
    return {"prediction": str(prediction[0])}
```

**Flow:**

1. COPY . /app
2. RUN pip install --no-cache-dir-r requirements.txt
3. EXPOSE 80
4. CMD ['uvicorn', 'app:app','--host', '0.0.0.0', '--port', '80']

