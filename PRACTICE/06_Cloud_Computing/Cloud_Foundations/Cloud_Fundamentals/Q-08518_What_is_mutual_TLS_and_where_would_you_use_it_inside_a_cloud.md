---
type: "practice"
difficulty: "Expert"
---

# What is mutual TLS and where would you use it inside a cloud service mesh?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Service Mesh](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Service_Mesh.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Mutual TLS (mTLS) is a security protocol that ensures both the client and server authenticate each other using X.509 certificates. In the context of a cloud service mesh, mTLS is crucial for securing communications between microservices by ensuring that only authorized parties can communicate and validating the identity of every node in the network. To understand where to use mutual TLS inside a cloud service mesh, let's first define what a service mesh is and then explore its security implications.

**Code:**

```
MERMAID
Copy
graph TB;
    A[Service Mesh] --> B[Ingress Gateway];
    B --> C[Sidecar Proxy];
    C --> D[Application Service];
```

**Flow:**

1. Client
2. mTLS Client Certificate
3. Service Mesh Ingress Gateway
4. (mTLS Server Authentication)
5. Service Mesh Sidecar Proxy
6. (Application Service)

