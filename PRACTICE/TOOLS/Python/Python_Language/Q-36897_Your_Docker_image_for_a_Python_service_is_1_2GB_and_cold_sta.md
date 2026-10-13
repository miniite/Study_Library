---
type: "practice"
difficulty: "Intermediate"
---

# Your Docker image for a Python service is 1.2GB and cold starts are slow. How would you slim it with multi-stage builds, wheels, and choosing a base image, without breaking native deps?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Admission Control and Load Shedding](<../../../../TOOLS/Python/Python_Language/Python_Tail_Latency/Admission_Control_and_Load_Shedding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a large Docker image size and slow cold starts for your Python service, we need to focus on optimizing the build process and selecting an appropriate base image. This can be achieved through multi-stage builds, using wheels, and choosing the right Python runtime environment. Here’s how you can approach this:

#### Overview
By employing multi-stage builds, we separate the initial dependencies installation from the final application deployment, reducing the overall image size. Using wheel distributions will help reduce package sizes further by avoiding recompilation of installed packages.

#### Triage

The question is asking how to optimize a Docker image for a Python service that is 1.2GB and experiences slow cold starts.
The interviewer is probing your understanding of Docker best practices, multi-stage builds, the use of wheels, and considerations for choosing base images.

#### Assumptions
include:
You have a basic understanding of Docker and its build process.
Your Python application has dependencies that can be installed in different stages.

#### First-principles Solution

Multi-Stage Builds

Multi-stage builds allow you to create an image with multiple layers, each representing a stage in the build process. The final stage includes only the necessary files, making the image smaller and faster to start.

DOCKERFILE
# Stage 1: Build environment
FROM python:3.9-slim-buster AS builder

# Install dependencies
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Stage 2: Production environment
FROM python:3.9-slim-buster

# Copy the built dependencies from the first stage
COPY --from=builder /app/. ./.

# Set up the application
WORKDIR /app
COPY . .

# Expose and run the service
EXPOSE 8080
CMD ["python", "your_app.py"]
Using Wheels

Wheels are precompiled Python distributions that include everything needed to install a package, making them smaller and faster than their source code counterparts. You can generate wheels for your dependencies using tools like pip wheel.

Ensure you have a requirements.txt file with all necessary packages.
Generate wheels:
SH
pip wheel -r requirements.txt --wheel-dir=./wheels
Update the Dockerfile to use these wheels:
DOCKERFILE
# Stage 1: Build environment
FROM python:3.9-slim-buster AS builder

# Install dependencies using wheels
WORKDIR /app
COPY wheels .
RUN pip install --no-cache-dir -r requirements.txt --find-links=./wheels

# Stage 2: Production environment
FROM python:3.9-slim-buster

# Copy the built dependencies from the first stage
COPY --from=builder /app/. ./.

# Set up the application
WORKDIR /app
COPY . .

# Expose and run the service
EXPOSE 8080
CMD ["python", "your_app.py"]
Choosing a Base Image

The python:3.9-slim-buster image is chosen for its small size (slim variant) and minimal footprint, which reduces the overall image size further.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

#### Common Mistakes
Failing to use multi-stage builds, not generating wheels, or choosing a base image that isn't optimized for production.

#### Follow-Ups
The interviewer might ask about performance trade-offs between different base images (e.g., Alpine vs. Debian) and how to further optimize the build process.

