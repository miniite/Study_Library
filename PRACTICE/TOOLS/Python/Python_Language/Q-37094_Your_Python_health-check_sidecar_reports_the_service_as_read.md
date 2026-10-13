---
type: "practice"
difficulty: "Intermediate"
---

# Your Python health-check sidecar reports the service as ready before dependencies are actually reachable, causing traffic to hit a broken pod. How would you separate liveness from readiness logic and probe real dependencies safely?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Graceful Shutdown](<../../../../TOOLS/Python/Python_Language/Graceful_Shutdown.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of reporting service readiness prematurely in Python sidecars, we need to separate the concepts of "liveness" and "readiness." Liveness checks ensure that a container is running and healthy, while readiness checks confirm that the container can serve requests. In this scenario, we want to ensure that our pod's dependencies are available before marking it as ready.

The key idea here is to implement a robust health-check mechanism that distinguishes between liveness and readiness probes effectively. Let's break down the problem step-by-step from first principles.

**Code:**

```python
# Example of Python liveness and readiness logic

import requests
from kubernetes import client, config

def check_liveness():
    # Simple health check to ensure the container is running
    try:
        response = requests.get('http://localhost/health')
        if response.status_code == 200:
            return True
    except Exception as e:
        print(f"Liveness check failed: {e}")
    return False

def check_readiness():
    # More complex health check to ensure dependencies are reachable
    try:
        # Example URL to a service dependency
        response = requests.get('http://service-dependency.example.com/health')
        if response.status_code == 200:
            print("Readiness check passed.")
            return True
    except Exception as e:
        print(f"Readiness check failed: {e}")
    return False

def main():
    liveness = check_liveness()
    readiness = check_readiness()

    if liveness and readiness:
        print("Both checks passed. Marking pod as ready.")
    else:
        print("One or more checks failed. Keeping pod not ready.")

if __name__ == "__main__":
    main()
```

**Example input / output:**

```text
Input:
Running the Python script with a Kubernetes liveness and readiness check.

Output:
Liveness check passed.
Readiness check failed: <Exception details here>
One or more checks failed. Keeping pod not ready.
```

**Flow:**

1. Health Check Initiation
2. Liveness Check
3. Liveness Passed
4. Readiness Check
5. Readiness Passed
6. Pod Marked Ready
7. Readiness Failed
8. Pod Not Ready

