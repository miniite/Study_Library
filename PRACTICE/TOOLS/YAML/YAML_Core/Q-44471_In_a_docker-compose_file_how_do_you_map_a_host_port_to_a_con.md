---
type: "practice"
difficulty: "Intermediate"
---

# In a docker-compose file how do you map a host port to a container port and what is the syntax gotcha with '80:80'?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Refactoring Helm Values](<../../../../TOOLS/YAML/YAML_Core/Refactoring_Helm_Values.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Docker Compose, mapping ports between your host machine and containers is essential for accessing services running inside the containers from outside. A common task involves exposing a container port to a host port using the ports section in the Docker Compose file. Let's dive into how you can map these ports correctly and understand some common gotchas.

TL;DR: To map a host port to a container port, use the format <host-port>:<container-port>. However, when specifying 80:80, be cautious of potential pitfalls related to service availability and precedence.

**Code:**

```yaml
version: '3.8'
services:
  web:
    image: nginx:latest
    ports:
      # Correct port mapping
      - "8080:80"
```

**Example input / output:**

```
Input:
YAML
Copy
version: '3.8'
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
Output: Nginx service running on host port 8080 and container port 80.
```

**Flow:**

1. Yes
2. No
3. Define Docker Compose File
4. Specify ports?
5. Use correct format<host-port>:<container-port>
6. Map 8080 to 80
7. Do not specify port mapping
8. Service accessible viahost:8080

