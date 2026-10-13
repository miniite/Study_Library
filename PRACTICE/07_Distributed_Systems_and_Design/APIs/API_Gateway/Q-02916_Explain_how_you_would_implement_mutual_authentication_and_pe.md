---
type: "practice"
difficulty: "Deep"
---

# Explain how you would implement mutual authentication and per-partner throttling for a B2B API where each partner has different SLAs and rate limits at the gateway.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Responsibilities and Anti-patterns](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Gateway_Responsibilities_and_Anti-patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Mutual authentication and per-partner throttling are critical components in securing and managing access to a B2B API Gateway, ensuring that only authorized partners can access the API while adhering to their specific service level agreements (SLAs). The key challenge is implementing these mechanisms efficiently at scale. In this scenario, as an experienced Backend Architect/Principal, I would approach the problem from first principles by breaking it down into its core components: mutual authentication, per-partner throttling, and ensuring compliance with SLAs.

Firstly, mutual authentication ensures that both parties are who they claim to be. This is crucial for B2B interactions as it prevents unauthorized access and eavesdropping. Secondly, per-partner throttling involves setting rate limits based on the specific SLAs agreed upon by each partner. This helps manage API usage and prevent abuse or overloading of services.

To implement these features effectively:

Mutual Authentication: We would use TLS mutual authentication to ensure that both the client (partner) and server (API Gateway) verify each other’s identities.
Per-Partner Throttling: We would configure rate limits dynamically based on pre-defined SLAs for each partner.
SLA Compliance: We would monitor API usage in real-time and enforce rate limits to ensure compliance with the agreed-upon SLAs.

Let's go through these components step-by-step:

Mutual Authentication

TLS mutual authentication involves establishing a secure connection where both the client and server present their certificates for verification. This can be achieved using a standard SSL/TLS handshake, but with mutual certificate validation enabled. Here’s an outline of how this could be implemented:

Client
Initiates TLS Handshake
C
Verifies Client's Certificate
Establishes Secure Connectionif Valid
Per-Partner Throttling

For per-partner throttling, we need to dynamically apply rate limits based on the partner’s SLA. This can be achieved by storing partner-specific configuration in a database or an API management system like Kong or Apicast.

Store Partner Configuration: Each partner would have its own entry in a database with details such as their allowed requests per minute (RPM).
Rate Limiting Logic: Implement rate limiting logic that checks the current request against the stored RPM for the partner. If the limit is exceeded, reject the request.

Here’s an example of how this could be structured:

No
Yes
Request
Check Partner Configuration
Is Rate Limit Exceeded?
Allow Request
Reject Request with 429 TooMany Requests
SLA Compliance Monitoring and Enforcement

Real-time monitoring of API usage is essential to enforce SLAs. We can use a combination of metrics, logs, and alerting mechanisms to ensure compliance.

Metrics Collection: Collect metrics on request counts, response times, and other relevant KPIs.
Alerting Mechanisms: Set up alerts based on these metrics to notify the team if any partner is exceeding their SLA.
Adjust Rate Limits as Needed: If a partner exceeds their SLA, adjust the rate limits dynamically to prevent further overuse.

#### Implementation
Details

For a robust and scalable implementation:

Use an API gateway like Kong or Apicast that supports both mutual authentication and dynamic rate limiting.
Leverage Kubernetes for managing service deployments and scaling resources based on real-time metrics.
Implement logging and monitoring tools (like Prometheus, Grafana) to continuously monitor API usage.

#### Common Mistakes
Overlooking Mutual Authentication: Failing to implement mutual TLS can lead to security vulnerabilities.
Inadequate Rate Limiting Logic: Not setting appropriate rate limits or failing to dynamically adjust them based on partner SLAs.
Poor Monitoring and Compliance Enforcement: Lack of real-time monitoring and alerting mechanisms can result in non-compliance with SLAs.

#### Closing Summary

Implementing mutual authentication and per-partner throttling at the API gateway level requires a comprehensive approach that covers security, compliance, and performance management. By leveraging modern tools and best practices, we can build a robust system that ensures secure, reliable, and compliant B2B interactions.

**Code:**

```python
# Example of TLS mutual authentication in Python using OpenSSL
import ssl

context = ssl.create_default_context(ssl.Purpose.CLIENT_AUTH)
context.load_cert_chain(certfile="path/to/cert.pem", keyfile="path/to/key.pem")
Tables
Partner	Allowed RPM
ABC	100
DEF	50
```

