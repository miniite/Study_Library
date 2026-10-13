---
type: "practice"
difficulty: "Deep"
---

# Design the gateway architecture to enforce an org-wide 'default-deny egress' where services can only call external APIs through a governed forward-proxy at the same edge; how do you reconcile north-south ingress and controlled egress in one platform without conflating their trust models?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Gateway Idempotency and Org Standards](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Gateway_Idempotency_and_Org_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce an org-wide 'default-deny egress' where services can only call external APIs through a governed forward-proxy at the same edge, we need to design an API Gateway that acts as both an ingress (north-south) and egress (east-west) control point. This approach ensures that all communication between internal services and external endpoints is managed and audited.

The key challenge is to reconcile these two trust models: the north-south traffic typically involves more relaxed controls since it's originating from trusted internal resources, while east-west traffic must be heavily restricted due to its direct access to external systems. We can achieve this by implementing a robust, layered architecture that separates concerns and enforces strict policies without conflating the ingress and egress trust models.

**Code:**

```
Egress Flow
Service Request to External API
API Gateway (Egress)
Governed Forward-Proxy
External API
Ingress Flow
User Request
API Gateway (Ingress)
Internal Service

(None)
```

