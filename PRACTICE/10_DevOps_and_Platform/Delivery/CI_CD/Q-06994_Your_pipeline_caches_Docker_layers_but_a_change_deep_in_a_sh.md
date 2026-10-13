---
type: "practice"
difficulty: "Intermediate"
---

# Your pipeline caches Docker layers but a change deep in a shared base layer invalidates almost everything and CI jumps to 25 minutes; how would you restructure Dockerfiles and layer ordering to maximize cache reuse?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ CI Caching](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When optimizing CI/CD pipelines, especially with Docker, we often need to balance between build time efficiency and cache utilization. In this scenario, a change in a shared base layer invalidates almost everything, leading to lengthy rebuilds. To address this issue, we can restructure our Dockerfiles and manage layers more effectively.

Firstly, let's understand what the problem is: a deep change in a shared base layer causes nearly all subsequent layers to be rebuilt, significantly increasing CI time from 25 minutes to an unacceptable level. The key idea here is to minimize the impact of changes on other layers by organizing Dockerfiles and layers properly.

To start, we should break down our application into smaller, more modular services or components. Each service can have its own Dockerfile, which helps in isolating changes and reducing the scope of rebuilds. Here’s how we might approach it:

Modularize Services: Create separate Dockerfiles for each microservice.
Optimize Layer Ordering: Ensure that non-critical layers come first and are shared across services.
Use Multi-Stage Builds: Leverage multi-stage builds to keep dependencies out of the final image, reducing cache invalidation.

Let's dive deeper into these steps:

Triaging the Problem

Modular Services:

Break down the application into smaller components (e.g., API, Database, UI).
Each component gets its own Dockerfile, which minimizes the impact of changes on other services.

Layer Ordering:

Critical layers that are less likely to change should come first.
Non-critical layers should follow and can be shared more widely.

Multi-Stage Builds:

Use multiple FROM statements in a single Dockerfile to build different stages, ensuring only necessary dependencies are included.

#### First-Principles Solution

Modularize Services:

PLAINTEXT
├── api
│   └── Dockerfile
├── db
│   └── Dockerfile
├── ui
│   └── Dockerfile

Each service has its own Dockerfile, reducing the impact of changes.

Optimize Layer Ordering:

PLAINTEXT
FROM base_image AS build
WORKDIR /app/api

COPY . .

# Critical layers first
RUN python setup.py install

# Non-critical layers
COPY config/ config/
COPY static/ static/

FROM base_image
COPY --from=build /app/api/dist/ dist/
WORKDIR /dist

This ordering ensures that critical dependencies are cached and reused, while non-critical files only need to be rebuilt when necessary.

Multi-Stage Builds:

PLAINTEXT
# API Dockerfile
FROM base_image AS build
COPY . .

RUN python setup.py install

FROM base_image
COPY --from=build /app/api/dist/ dist/
WORKDIR /dist

# UI Dockerfile
FROM base_image AS build
COPY . .

RUN npm install

FROM base_image
COPY --from=build /app/ui/dist/ dist/
WORKDIR /dist

#### Closing Summary
and Common Mistakes/Follow-Ups

By modularizing services, optimizing layer ordering, and leveraging multi-stage builds, we can significantly reduce the impact of changes on our CI/CD pipeline. This approach not only minimizes cache invalidation but also improves build time efficiency.

#### Common Mistakes

Not breaking down the application into smaller components.
Poor layer ordering leading to frequent rebuilds.
Overlooking the use of multi-stage builds, which can greatly enhance caching and reduce build times.

For follow-ups, consider discussing how to handle shared libraries or dependencies across services, and how to further optimize network and resource usage in your CI/CD pipeline.

**Comparison:**

```
Layer	Type	Description
1	Critical	Core dependencies and setup
2	Non-Critical	Additional configuration and assets
```

**Code:**

```
DOCKER
Copy
# API Dockerfile example
FROM base_image AS build
COPY . .
RUN python setup.py install

FROM base_image
COPY --from=build /app/api/dist/ dist/
WORKDIR /dist

# UI Dockerfile example
FROM base_image AS build
COPY . .
RUN npm install

FROM base_image
COPY --from=build /app/ui/dist/ dist/
WORKDIR /dist
```

**Data:**

```yaml
services:
  - api
  - db
  - ui
dependencies:
  - python-library
  - node-modules
```

**Flow:**

1. Base Image
2. (build stage)
3. Copy source code
4. Run setup.py install
5. E1
6. Copy built app to final image
7. Base Image
8. Copy from build stage
9. (app/dist directory)

