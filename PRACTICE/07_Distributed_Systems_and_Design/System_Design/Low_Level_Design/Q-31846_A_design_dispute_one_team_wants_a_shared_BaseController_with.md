---
type: "practice"
difficulty: "Deep"
---

# A design dispute: one team wants a shared "BaseController" with lots of hooks; another argues it becomes a god-base-class over time; as principal, how do you decide between inheritance-based framework hooks and composition/middleware, and what do you standardize?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Composition vs Inheritance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Composition_vs_Inheritance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a system, particularly in a backend architecture context, one of the key decisions is whether to use inheritance via a shared "BaseController" with many hooks or to opt for composition and middleware. As a Backend Architect/Principal, my approach involves evaluating both approaches from first principles, considering trade-offs, and making strategic decisions based on the system's requirements and future scalability.

In this scenario, one team advocates for a shared BaseController filled with various hooks, while another argues that such an approach can lead to a "god base class" issue over time. The crux of the debate is whether inheritance or composition/middleware offers better maintainability, flexibility, and scalability in the long run.

To make this decision, we need to consider several factors:

Maintainability: A highly coupled BaseController can become difficult to modify and extend without breaking existing code.
Flexibility: Composition allows for more modular design where components can be swapped or added independently.
Scalability: Inheritance-based designs can lead to a complex class hierarchy, whereas composition scales better by allowing the addition of new functionality in a less intrusive manner.
Risk Management: A monolithic base controller increases the risk of introducing bugs and makes debugging more challenging.
Step-by-Step Reasoning

Understand the System's Requirements:

What are the primary use cases? Are there specific functionalities that need to be applied consistently across controllers?
How complex is the system likely to become over time?

Evaluate Inheritance-based Framework Hooks:

#### Pros
Can provide a standardized starting point for all controllers, simplifying implementation.

#### Cons
Tight coupling can make it difficult to change or extend the base behavior without affecting derived classes.

Evaluate Composition and Middleware:

#### Pros
Loose coupling between components allows for more flexible and maintainable designs.

#### Cons
Requires a different approach to structuring code, potentially adding complexity during initial implementation.
Standardization

Given the depth of understanding required, I would standardize on composition over inheritance where possible. This approach aligns with modern design principles such as SOLID, particularly the Single Responsibility Principle (SRP) and Open/Closed Principle (OCP). Composition allows for clear separation of concerns and easier testing.

Here’s a high-level rationale:

Composition: Encapsulate common functionalities within separate modules or services that can be injected into controllers. This keeps each controller focused on its primary responsibility.
Middleware: Use middleware to handle cross-cutting concerns such as logging, authentication, and error handling. Middleware can be added or removed without modifying the core logic of the controller.

#### Example
Middleware vs Inheritance

#### Cons
ider a basic HTTP request lifecycle:

PLAINTEXT
+-------------------+
|  Router          |
|   (Handles URL)  | <------> +--------------+
|                  |         |  Controller |
+-------------+----+         |     (Handle business logic)|
                            +--------------+
                                 |
                         +--------v---------+
                         |   Middleware    |
                         |  (Authentication,|
                         |   Logging, etc.)  |
                         +-----------------+

In this setup:

The Router handles the request routing.
The Controller deals with the core business logic.
Middlewares are used for common tasks like authentication and logging.

#### Conclusion

By standardizing on composition over inheritance-based hooks, we promote a more modular and maintainable design. This approach aligns with modern software engineering practices and ensures that our system remains flexible and scalable as it evolves.

#### Common Mistakes

Overusing Inheritance: Using too many levels of inheritance can lead to complex and hard-to-maintain hierarchies.
Lack of Modularity: Failing to break down responsibilities into smaller, reusable components.
Ignoring Composition: Overlooking the benefits of composition in favor of simple inheritance.

**Comparison:**

```
Inheritance vs Composition	Pros (Composition)	Cons (Composition)	Pros (Inheritance)	Cons (Inheritance)
Coupling	Low	High	High	Varies
Testability	High	Moderate	Moderate	Low
Scalability	High	Moderate	Varies	Varies
Flexibility	High	Moderate	Moderate	Low
```

**Code:**

```python
# Example of a Middleware pattern

class AuthenticationMiddleware:
    def __init__(self, handler):
        self.handler = handler

    def __call__(self, request):
        # Check if user is authenticated
        if not request.user.is_authenticated:
            raise PermissionError("Unauthorized")

        return self.handler(request)

# Example usage in a Controller

class UserController:
    def __init__(self, auth_middleware: AuthenticationMiddleware):
        self.auth_middleware = auth_middleware

    @staticmethod
    def handle_request(request):
        # Handle user-related logic
        pass

user_controller = UserController(AuthenticationMiddleware(UserController.handle_request))
```

**Example input / output:**

```text
Input:
- URL: /user/profile
- Method: GET
- Headers: Authorization: Bearer token123

Output:
- Status Code: 401
- Message: Unauthorized
```

**Data:**

```json
{
  "request": {
    "url": "/user/profile",
    "method": "GET",
    "headers": {
      "Authorization": "Bearer token123"
    }
  },
  "response": {
    "status_code": 401,
    "message": "Unauthorized"
  }
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Router
5. Is Request Valid?
6. Middleware (Authentication,Logging)
7. Controller
8. Process Business Logic?
9. Return Response
10. Middleware (Logging, ErrorHandling)
11. Controller
12. Return Response

