---
type: "practice"
difficulty: "Deep"
---

# You're the reviewer on a design that puts all business logic in database stored procedures for 'performance' — probe it: what does that do to testability deploy-safety and horizontal scaling and how would you re-architect the hot path?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Audit and Compliance Platforms](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Audit_and_Compliance_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are evaluating a design decision where all business logic is encapsulated in database stored procedures, justified primarily for performance reasons. This approach has significant implications on testability, deployment safety, and horizontal scaling. To properly re-architect the "hot path," it's crucial to understand these trade-offs and explore alternative designs that balance these concerns.

The core idea here is to critically assess why putting all business logic in stored procedures might be problematic and propose a more scalable, maintainable solution. Stored procedures can indeed provide performance benefits by reducing network latency; however, this comes at the cost of reduced testability, deployment safety, and scalability.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Application Layer] --> B[Database];
    B --> C[Stored Procedures];
    D[API Gateway] --> E[DB Connection Pool];
```

**Flow:**

1. Initial Request
2. Application Layer Code
3. C
4. Database Connection Pool
5. Stored Procedure Execution
6. Query/Transaction Results
7. Response to Client

