---
type: "practice"
difficulty: "Deep"
---

# How would you handle a downstream service returning a 500 to your REST API without cascading the failure to your client?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ HTTP Status Codes](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Status_Codes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When handling a situation where a downstream service returns a 500 Internal Server Error (or any other non-2xx status code) in a REST API, it’s crucial to ensure that we do not simply pass this error directly to the client. Instead, our goal is to handle and potentially mask the failure gracefully while providing meaningful feedback to both internal operations and clients.

The question is probing your understanding of how to manage complex service-to-service interactions without compromising user experience or exposing sensitive information. Here’s a step-by-step breakdown from first principles:

Understand the Context: In a distributed system, each microservice may have its own error handling policies.
Define Error Handling Policies: Determine what types of errors are acceptable and which ones should be masked.
Implement Centralized Logging and Monitoring: Ensure that all errors are logged and monitored for better visibility.
Decide on the Response: Choose a response that is both informative to the client and non-disclosing.

