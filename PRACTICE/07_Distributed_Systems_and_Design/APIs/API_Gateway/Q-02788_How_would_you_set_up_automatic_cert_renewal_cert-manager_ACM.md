---
type: "practice"
difficulty: "Intermediate"
---

# How would you set up automatic cert renewal (cert-manager/ACME) for the gateway's public TLS, and what's your alerting so you never wake up to an expired cert 500ing every request?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Responsibilities and Anti-patterns](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Gateway_Responsibilities_and_Anti-patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure our API Gateway always has a valid SSL/TLS certificate, we'll set up automatic renewal using cert-manager and ACME. This involves configuring cert-manager to issue certificates via the Let's Encrypt CA, setting up a TLS secret in Kubernetes, and integrating with the API Gateway. Additionally, we need robust alerting so that we're notified before any downtime due to an expired certificate.

**Code:**

```

apiVersion: cert-manager.io/v1 kind: Issuer metadata: name: letsencrypt-prod spec: acme: email: admin@example.com server: https://acme-v02.api.letsencrypt.org/directory privateKeySecretRef: name: example-issuer-private-key solvers: - http01: ingress: class: nginx

CODE
Copy

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-certificate
spec:
  secretName: example-tls-secret
  issuerRef:
    name: letsencrypt-prod
    kind: Issuer
  dnsNames:
  - api.example.com
YAML
Copy
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  tls:
  - hosts:
    - api.example.com
    secretName: example-tls-secret
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

**Example input / output:**

```text
No specific input-output examples are needed, but the configuration above is a typical workflow to set up cert-manager and Ingress.
```

**Flow:**

1. API Gateway Ingress
2. cert-manager
3. Lets Encrypt CA
4. TLS Certificate Issuance
5. TLS Secret Created inKubernetes
6. Ingress Setup with TLS Secret

