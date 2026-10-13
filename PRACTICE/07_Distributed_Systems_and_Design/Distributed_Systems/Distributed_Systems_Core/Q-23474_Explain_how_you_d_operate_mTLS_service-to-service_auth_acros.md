---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd operate mTLS service-to-service auth across a few hundred microservices with a mesh. How do you handle cert rotation at scale and contain blast radius when one service identity is compromised?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Multi-tenant vs Dedicated Clusters](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Multi-tenant_vs_Dedicated_Clusters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To operate mutual TLS (mTLS) service-to-service authentication in a distributed system with hundreds of microservices, we need to ensure secure communication between services while managing certificates efficiently and securely. Key challenges include handling certificate rotations at scale and minimizing the blast radius when an identity is compromised. The approach involves using a centralized Certificate Authority (CA), leveraging Service Mesh capabilities, and implementing robust monitoring and logging.

**Code:**

```bash
# Example CA setup
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ca.key -out ca.crt

# Example service certificate generation
openssl req -new -key server.key -out server.csr
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365
```

**Data:**

```yaml
# Sample CA configuration
ca:
  key: |
    -----BEGIN PRIVATE KEY-----
    ...
    -----END PRIVATE KEY-----
  cert: |
    -----BEGIN CERTIFICATE-----
    ...
    -----END CERTIFICATE-----

# Service certificate request template
service-csr-template:
  CN: {{.ServiceName}}
  DNSName: [{{.ServiceName}}-internal.example.com]
```

**Flow:**

1. Generate CSR
2. Sign with CA
3. Issue Signed Certificate
4. Service uses mTLS for internalcommunication
5. Centralized Monitoring &Logging
6. Monitor for CompromisedIdentities

