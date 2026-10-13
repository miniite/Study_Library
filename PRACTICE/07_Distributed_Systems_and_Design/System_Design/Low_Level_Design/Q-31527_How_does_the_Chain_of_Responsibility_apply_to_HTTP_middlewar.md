---
type: "practice"
difficulty: "Intermediate"
---

# How does the Chain of Responsibility apply to HTTP middleware and where does next() fit?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Lock Granularity Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Lock_Granularity_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The Chain of Responsibility design pattern is a powerful way to handle requests in a decoupled manner, making it an excellent fit for managing HTTP middleware in web applications. This pattern allows different modules or components to handle a request without needing to know the structure or context of who handles subsequent steps. In this answer, we'll explore how next() fits into this design and discuss its application through a detailed first-principles solution.

The Chain of Responsibility pattern essentially creates a linked list of objects that are responsible for processing requests. Each object has a reference to the next object in the chain. If an object cannot handle a request, it forwards the request to the next object until someone in the chain can process it or a default handler takes over.

In the context of HTTP middleware, each middleware function could be treated as one node in the chain. The next() function acts as a pass-through mechanism that links these nodes together, ensuring that requests are processed sequentially but with flexibility and loose coupling between components.

**Code:**

```

class Middleware: def init(self): self.next_middleware = None

CODE
Copy
def set_next(self, middleware):
    self.next_middleware = middleware

def handle_request(self, request):
    if self.next_middleware is not None:
        return self.next_middleware.handle_request(request)
    else:
        # Default handler or error handling
        print("Request not handled by any middleware.")
CODE
Copy

class RequestHandler(Middleware):
    def __init__(self, func):
        super().__init__()
        self.func = func

    def handle_request(self, request):
        result = self.func(request)
        if result is None:
            # Forward to the next middleware
            return self.next_middleware.handle_request(request)

# Example usage
def log_request(request):
    print("Request logged:", request)
    return "Logged"  # Return value indicates if handled

middleware1 = RequestHandler(log_request)
middleware2 = RequestHandler(lambda req: f"Processed {req}")
middleware3 = RequestHandler(lambda req: f"Final response for {req}")

middleware1.set_next(middleware2)
middleware2.set_next(middleware3)

response = middleware1.handle_request("Sample request")
print(response)
```

**Example input / output:**

```text
Request logged: Sample request
Processed Sample request
Final response for Sample request
```

**Flow:**

1. mermaid
2. graph LR;
3. A[Middleware1] --> B[Middleware2];
4. B --> C[Middleware3];
5. C --> D[HandleRequestDefault];
6. E[Request] --> A;

