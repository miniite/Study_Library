---
type: "practice"
difficulty: "Intermediate"
---

# How would you install Python dependencies inside a Docker image, and why is copying requirements.txt before the app code a common best practice?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Compressed File Handling](<../../../../TOOLS/Python/Python_Language/Compressed_File_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To install Python dependencies inside a Docker image, you typically use pip to install from a requirements.txt file that lists all your project's dependencies. This method is widely used because it ensures reproducibility and consistency across different environments. By copying the requirements.txt before the application code, we avoid potential version conflicts or missing dependencies.

First, let’s break down what the question is asking:

How to install Python dependencies in a Docker image - We need to use Docker's filesystem capabilities to copy files into the container and then run commands.
Why requirements.txt is commonly used before application code - This is about best practices, so we’ll discuss why this approach enhances reliability and maintainability.

#### Triage

The question is essentially asking for a step-by-step process of setting up Python dependencies in Docker and explaining the rationale behind placing requirements.txt early. We will assume basic familiarity with Dockerfile syntax and use Python 3 as our example.

#### First-Principles Solution
Step 1: Define the Dockerfile

Start by creating a simple Dockerfile that sets up your environment, copies files into the image, and installs dependencies.

DOCKERFILE
# Use an official Python runtime as a parent image
FROM python:3.9-slim-buster

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
CMD ["python", "app.py"]
Step 2: Build and Run the Docker Image

Use docker build to create your image, ensuring that all dependencies are installed correctly.

BASH
docker build -t my-python-app .

Then run the image with:

BASH
docker run -p 4000:80 my-python-app

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, placing requirements.txt before application code in a Dockerfile ensures that all dependencies are installed correctly and consistently. This is particularly important for multi-container deployments or when you need to share images across different environments.

#### Common Mistakes
Forgetting to update the requirements.txt file: Ensuring it always contains up-to-date package versions.
Not using --no-cache-dir with pip install: This can cause issues if dependencies are already installed and might lead to unexpected behavior.
Ignoring environment variable handling: Ensure that any environment variables required by your application are set correctly.

#### Follow-Up
Questions

The interviewer might ask:

What would you do if the project uses virtual environments?
How would you manage dependencies for a large Python project with multiple packages and versions?

**Comparison:**

| Step | Action | Explanation |
|------|--------|-------------|
| 1    | Define Dockerfile | Sets up Python environment and installs dependencies |
| 2    | Build image       | Uses `docker build` command with `-t my-python-app` tag |
| 3    | Run container     | Maps host port to container port using `-p 4000:80` |

**Code:**

```dockerfile
# Example Dockerfile snippet
FROM python:3.9-slim-buster
WORKDIR /app
COPY . /app
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 80
ENV NAME World
CMD ["python", "app.py"]
```

**Example input / output:**

```text
# Building the Docker image
$ docker build -t my-python-app .
[...]
Successfully built e960a3e4b05f

# Running the Docker container
$ docker run -p 4000:80 my-python-app
* Serving Flask app "app.py" (lazy loading)
* Environment: production
  WARNING: This is a development server. Do not use it in a production deployment.
  Use a production WSGI server instead.
* Debug mode: off
* Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
```

**Data:**

```json
// Sample requirements.txt file
flask==1.1.2
click==7.1.2
```

**Flow:**

1. Build Dockerfile
2. COPY . /app
3. RUN pip install --no-cache-dir-r requirements.txt
4. EXPOSE 80
5. CMD ['python', 'app.py']

