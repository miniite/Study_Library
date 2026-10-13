---
type: "practice"
difficulty: "Basic"
---

# You're asked to add a maintenance-mode page at the gateway for planned DB downtime; how do you serve a static response for write routes while keeping read routes live, and roll it back cleanly?

**Difficulty:** Basic · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Resilience to Dependencies](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Resilience_to_Dependencies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To implement a maintenance mode in an API Gateway for planned database downtime, we need to serve a static 503 response for write operations while allowing read operations to continue uninterrupted. This involves configuring route rules and possibly using custom middleware or HTTP status codes.

To achieve this, let's break down the problem from first principles: understanding the current routing logic, implementing the maintenance mode, and ensuring a clean rollback process.

#### Triage
The interviewer is probing your understanding of API Gateway configurations, routing logic, and how to selectively disable certain operations while keeping others functional. We'll need to make sure that write routes (e.g., POST, PUT) are redirected to a static 503 response page, whereas read routes (e.g., GET, HEAD) continue to function as normal.

Step-by-Step Solution:

Identify Write and Read Routes: First, we need to understand the HTTP methods used for write operations (like POST or PUT) versus read operations (like GET or HEAD).
Configure API Gateway Rules: Use the API Gateway's routing rules to differentiate between these operation types.
Implement Maintenance Logic: For write routes, serve a static 503 response with an appropriate message explaining the maintenance period.
Ensure Read Routes Remain Active: Ensure that read operations are not affected and continue to function normally.

Here’s how we can implement this in practice:

Identify Write and Read Methods:

Write methods: POST, PUT
Read methods: GET, HEAD

Configure API Gateway Rules:

Create two sets of rules:
One set for write operations (POST, PUT) that will redirect to a maintenance page.
Another set for read operations (GET, HEAD) that remain unchanged.

Implement Maintenance Logic:

For the rule targeting write methods, we can use custom middleware or rewrite the URL to point to a static 503 response file.

#### Example
configuration in an API Gateway like AWS API Gateway:
PLAINTEXT
# Rule for write operations (POST/PUT)
if http.method is "POST" or http.method is "PUT":
    rewrite.response.body = "Maintenance Mode: Write operations are currently unavailable."
    return 503

# Rule for read operations (GET/HEAD)
if http.method is "GET" or http.method is "HEAD":
    pass

Clean Rollback Process:

When the maintenance period ends, revert the API Gateway rules back to their original state.
Ensure that all write routes return to normal operation and any static 503 responses are removed.

#### Closing Summary
By carefully configuring our API Gateway rules, we can selectively disable specific operations (write methods) while keeping others (read methods) operational. This ensures minimal disruption during planned downtime.

#### Common Mistakes

Forgetting to configure read routes properly, leading to unexpected downtime for users.
Not having a clear rollback plan, which could lead to prolonged maintenance periods.

Step-by-Step Code Example:

PLAINTEXT
# Example API Gateway Rule Configuration (AWS API Gateway)
writeRule:
    http.method:
        - POST
        - PUT
    action:
        rewrite.response.body: "Maintenance Mode: Write operations are currently unavailable."
        set.status.code: 503

readRule:
    http.method:
        - GET
        - HEAD
    action:
        pass

#### Conclusion
By following these steps, we can effectively manage maintenance periods in our API Gateway without causing unnecessary downtime for users.

