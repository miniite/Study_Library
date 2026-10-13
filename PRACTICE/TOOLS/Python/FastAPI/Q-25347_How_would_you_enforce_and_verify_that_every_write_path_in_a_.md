---
type: "practice"
difficulty: "Deep"
---

# How would you enforce and verify that every write path in a FastAPI service emits an audit log, and how do you keep that guarantee as new endpoints are added by other engineers?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Real-time and Batch Inference APIs](<../../../../TOOLS/Python/FastAPI/Real-time_and_Batch_Inference_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that every write path in a FastAPI service emits an audit log, we need to implement a robust logging mechanism that is decoupled from individual endpoints and can be easily enforced across the entire application. This involves both design choices at the architecture level and implementation strategies.

The core idea here is to leverage middleware or decorators to capture and log every write operation uniformly, ensuring that this requirement remains intact as new endpoints are added by other engineers. We will break down the solution step-by-step from first principles.

First, let's understand what we're aiming for: an audit log ensures that every write (e.g., create, update, delete) action is recorded and traceable. This not only helps in debugging but also in compliance with regulatory requirements.

#### Triaging the Question

The question essentially probes how to maintain consistency across a fast-growing application where new endpoints might be added by multiple engineers without breaking the audit log requirement. We need to ensure that all write operations are logged, regardless of where they occur within the service.

Key Assumptions and Context
FastAPI Service: The backend service is built using FastAPI, a modern, fast (high-performance) web framework for building APIs with Python 3.7+.
Write Operations: We focus on operations like creating, updating, or deleting data.
Audit Logging: Every write operation should be logged in a consistent format.

#### First-Principles Solution

To enforce audit logging across all write paths, we can leverage FastAPI's middleware capabilities. Middleware provides a powerful way to intercept requests and responses at various stages of the request lifecycle. We will create a custom middleware that will inspect HTTP methods and log appropriate actions.

Step 1: Define Custom Middleware

We'll define a custom middleware function that will check if an operation is a write action (e.g., POST, PUT, DELETE) and then record it in our audit logs.

PYTHON
from fastapi import FastAPI, Request
from starlette.middleware.base import BaseHTTPMiddleware
import logging

app = FastAPI()

class AuditLoggerMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        if "body" in await request.body():
            # Check for write actions (POST, PUT, DELETE)
            operation = request.method
            data = await request.json()  # Assuming JSON body

            logging.info(f"Audit Log - {operation} Operation: {data}")

        response = await call_next(request)

        return response

# Apply middleware to the FastAPI app
app.add_middleware(AuditLoggerMiddleware)
Step 2: Ensure Middleware Applies Globally

By adding this middleware to our FastAPI application, it will automatically intercept all incoming requests and log write operations. This ensures that no new endpoint can bypass audit logging without explicitly opting out.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By leveraging custom middleware, we ensure that every write path in the service is logged uniformly. New engineers adding endpoints only need to be aware of the existing middleware setup, simplifying maintenance and ensuring consistency. However, it’s crucial to also consider:

Performance Impact: Logging can introduce overhead, especially for high-throughput services.
Security: Ensure that sensitive information in audit logs is properly sanitized or masked.

#### Common mistakes
include:

Not applying the middleware globally, leading to inconsistent logging.
Failing to sanitize data before logging, potentially exposing sensitive information.
Overlooking performance implications of extensive logging.

For follow-ups, you might discuss how to balance logging detail with performance and security concerns, as well as integrating more sophisticated logging frameworks like ELK Stack for larger-scale deployments.

