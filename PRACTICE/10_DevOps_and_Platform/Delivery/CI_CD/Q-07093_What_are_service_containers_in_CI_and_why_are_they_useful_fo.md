---
type: "practice"
difficulty: "Intermediate"
---

# What are service containers in CI and why are they useful for backend tests?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Integration Testing and Test Doubles](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Integration_Testing_and_Test_Doubles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Service containers in CI (Continuous Integration) are isolated environments that simulate production-like conditions, allowing developers to run automated backend tests reliably. They ensure consistency across different development, testing, and deployment stages by providing a standardized runtime environment.

#### Triage

The question is asking about the concept of service containers within CI processes and their utility for backend testing. The key aspects being probed include understanding the purpose, benefits, and implementation details of containerization in this context. Common assumptions are that we're discussing Docker containers, but other container technologies could also be applicable.

Step-by-Step Solution
Definition: Service containers are lightweight, standalone packages that include everything needed to run a backend service, including code, runtime, system tools, libraries, and settings.
Purpose in CI: Containers help ensure that the backend services behave as expected in different environments without relying on specific local configurations or dependencies.
Benefits:
Isolation: Each container runs in its own environment, preventing conflicts with other processes.

#### Cons
istency: Ensures that tests and deployments are consistent across all stages.
Reproducibility: Facilitates easier debugging and issue resolution by providing a known state for testing.
Speed: Faster deployment cycles due to reduced dependency issues.

#### Implementation
Dockerfile:
DOCKERFILE
# Example Dockerfile snippet
FROM node:14

WORKDIR /app

COPY package.json .
COPY yarn.lock .

RUN yarn install --frozen-lockfile

COPY . .

EXPOSE 3000

CMD ["yarn", "start"]
Docker Compose:
YAML
version: '3'
services:
  backend:
    build: ./backend
    ports:
      - "3000:3000"
    networks:
      - mynetwork
    depends_on:
      - db

  db:
    image: postgres:12
    environment:
      POSTGRES_USER: testuser
      POSTGRES_PASSWORD: testpass
    volumes:
      - ./data/pgdata:/var/lib/postgresql/data
    networks:
      - mynetwork

networks:
  mynetwork:
CI/CD Pipeline (using GitHub Actions):
YAML
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:

    runs-on: ubuntu-latest

    strategy:
      matrix:
        python-version: [3.8, 3.9]

    steps:
    - uses: actions/checkout@v2
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v2
      with:
        python-version: ${{ matrix.python-version }}
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: Build and Test Docker image
      run: docker build -t my-backend .
    - name: Run tests
      env:
        DB_HOST: db  # Service container name from docker-compose.yml
      run: docker compose up -d --build backend && docker exec -it $(docker ps -aq -f name=my-backend) sh -c "pytest"

#### Closing Summary
and Common Mistakes/Follow-Ups

Service containers in CI provide a robust environment for running backend tests. They ensure consistency, isolation, and reproducibility across different stages of the development lifecycle. Common mistakes include not properly defining dependencies or ignoring network configuration issues between services.

#### Follow-up
questions might explore specific container orchestration tools (like Kubernetes) or strategies for scaling test environments.

**Code:**

```
Example Dockerfile snippet

FROM node:14

WORKDIR /app

COPY package.json . COPY yarn.lock .

RUN yarn install --frozen-lockfile

COPY . .

EXPOSE 3000

CMD ["yarn", "start"]

CODE
Copy

```yaml
# Example docker-compose.yml
version: '3'
services:
  backend:
    build: ./backend
    ports:
      - "3000:3000"
    networks:
      - mynetwork
    depends_on:
      - db

  db:
    image: postgres:12
    environment:
      POSTGRES_USER: testuser
      POSTGRES_PASSWORD: testpass
    volumes:
      - ./data/pgdata:/var/lib/postgresql/data
    networks:
      - mynetwork

networks:
  mynetwork:
YAML
Copy
# Example GitHub Actions CI/CD configuration
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:

    runs-on: ubuntu-latest

    strategy:
      matrix:
        python-version: [3.8, 3.9]

    steps:
    - uses: actions/checkout@v2
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v2
      with:
        python-version: ${{ matrix.python-version }}
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: Build and Test Docker image
      run: docker build -t my-backend .
    - name: Run tests
      env:
        DB_HOST: db  # Service container name from docker-compose.yml
      run: docker compose up -d --build backend && docker exec -it $(docker ps -aq -f name=my-backend) sh -c "pytest"
```

