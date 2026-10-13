---
type: "practice"
difficulty: "Intermediate"
---

# How would you specify the exact versions of scikit-learn and other ML libraries so a FastAPI model server reproduces training-time behavior?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Model Consistency Across Deploys](<../../../../TOOLS/Python/FastAPI/Model_Consistency_Across_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting up a FastAPI model server, ensuring reproducibility is crucial, especially when dealing with machine learning models that rely on specific versions of libraries like scikit-learn. This involves specifying exact library versions to avoid issues stemming from different environments or dependency updates.

#### Triaging the Question

The question asks about specifying exact versions of scikit-learn and other ML libraries in a FastAPI model server context, aiming to ensure that training-time behavior is reproducible. The key points are:

Library Version Consistency: Ensuring all dependencies use the same version used during the training phase.
Environment Isolation: Using tools like Docker to create an isolated environment.

#### First-Principles Solution

To achieve this, follow these steps:

Step 1: Define Exact Dependencies in requirements.txt

Create a requirements.txt file that lists all your dependencies with exact versions. For example:

PLAINTEXT
numpy==1.20.3
scikit-learn==0.24.2
pandas==1.2.5
Step 2: Use Docker for Environment Isolation

Use a Dockerfile to create an environment that matches the development or production environment exactly. Here is a basic example of a Dockerfile:

DOCKERFILE
# Using a base image
FROM python:3.8-slim

# Install necessary packages
RUN apt-get update && apt-get install -y \
    git \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install dependencies from requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Expose port 8000 for FastAPI server
EXPOSE 8000

# Command to run your app
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
Step 3: Build and Run the Docker Container

Build the Docker image using:

BASH
docker build -t my-fastapi-server .

Then run it with:

BASH
docker run -p 8000:8000 my-fastapi-server

#### Closing Summary
+ Common Mistakes/Follow-Ups

By specifying exact library versions and using Docker, you ensure that your FastAPI model server runs in an environment identical to the one used during training. This approach helps prevent issues like version mismatches or unexpected behavior due to dependency updates.

#### Common Mistakes

Failing to Specify Exact Versions: Not including exact package versions can lead to different behaviors across environments.
Not Using Environment Isolation: Without Docker, it's harder to guarantee consistent runtime environments.
Trialing Common Follow-Ups

How do you manage environment differences between development and production?

Use CI/CD pipelines with pre-defined environments that match your production setup.

What if new versions of libraries are released frequently?

Periodically update the requirements.txt based on library release notes, but ensure a smooth transition.

**Code:**

```dockerfile
# Example Dockerfile content
FROM python:3.8-slim

RUN apt-get update && apt-get install -y \
    git \
    curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY . /app

RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Example input / output:**

```text
# Commands for building and running the Docker container
docker build -t my-fastapi-server .
docker run -p 8000:8000 my-fastapi-server
```

**Data:**

```plaintext
# Example requirements.txt content
numpy==1.20.3
scikit-learn==0.24.2
pandas==1.2.5
```

