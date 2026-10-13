---
type: "practice"
difficulty: "Intermediate"
---

# How do you design a FastAPI health and readiness probe for an LLM gateway that distinguishes 'process alive' from 'the provider and vector store are actually reachable'?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Batching LLM Requests](<../../../../TOOLS/Python/FastAPI/Batching_LLM_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust FastAPI health and readiness probe for an LLM (Language Model) gateway, we need to distinguish between two states: "process alive" and "the provider and vector store are actually reachable." This distinction is crucial for ensuring that the service is not only running but also functional. Let's break this down step-by-step.

First, triage the question:

We're designing a FastAPI endpoint for health checks.
The goal is to differentiate between two states: process availability and full functionality.
We need to consider how to implement these probes effectively.

Now, let's reason from first principles:

1. Define the Health Probes
Process Alive

This state indicates that the FastAPI service itself is running but may not be fully operational. For example, network services might be down or some dependencies might be missing.

Readiness Probe

This state confirms whether the LLM gateway can actually communicate with the underlying language model provider and vector store. This means the health probe should check if the language model API responses are valid and the vector store is accessible.

2. Implement the FastAPI Endpoints

We'll create two endpoints: one for checking process status (/health) and another for readiness status (/readiness).

Process Alive Health Check
PYTHON
from fastapi import APIRouter, HTTPException
import uvicorn

router = APIRouter()

@router.get("/health")
async def health_check():
    return {"status": "Process alive"}
Readiness Probe

To implement the readiness probe, we need to perform actual checks against the LLM provider and vector store.

PYTHON
from fastapi import APIRouter, HTTPException
import requests

router = APIRouter()

@router.get("/readiness")
async def readiness_probe():
    try:
        # Check if the language model API is accessible
        response_model = requests.get("http://model-provider-endpoint/api/health", timeout=5)

        if not response_model.status_code == 200:
            raise HTTPException(status_code=response_model.status_code, detail="Model provider is unreachable")

        # Check if the vector store is accessible
        response_vector_store = requests.get("http://vector-store-endpoint/api/health", timeout=5)

        if not response_vector_store.status_code == 200:
            raise HTTPException(status_code=response_vector_store.status_code, detail="Vector store is unreachable")

        return {"status": "Readiness check passed"}
    except Exception as e:
        raise HTTPException(status_code=503, detail=f"Readiness check failed: {str(e)}")
3. Configure the Probes in Kubernetes (Optional)

If you are deploying this service on a container orchestration platform like Kubernetes, you can configure these probes directly.

YAML
apiVersion: v1
kind: Pod
metadata:
  name: llm-gateway-pod
spec:
  containers:
    - name: llm-gateway
      image: your-image
      readinessProbe:
        httpGet:
          path: /readiness
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 10
      livenessProbe:
        httpGet:
          path: /health
          port: 80
        initialDelaySeconds: 15
        periodSeconds: 30

#### Closing Summary
and Common Mistakes
Common Mistake: Failing to distinguish between process availability and service readiness, leading to a false sense of security.
Key Point: Always perform actual checks (e.g., HTTP requests) to ensure the underlying services are up and running.

By implementing these steps, you can ensure that your LLM gateway is both available and ready for use. This approach provides clear visibility into the health status of your service and helps in troubleshooting issues effectively.

