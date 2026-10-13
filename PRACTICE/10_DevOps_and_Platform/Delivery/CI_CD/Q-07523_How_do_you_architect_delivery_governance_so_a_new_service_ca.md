---
type: "practice"
difficulty: "Expert"
---

# How do you architect delivery governance so a new service can't reach prod for the first time without a rollback path, health/version endpoints, authz on every route, and an owning on-call team wired in, enforced as an automated 'production-readiness gate' rather than a checklist someone signs?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Safe Delivery Platform Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Safe_Delivery_Platform_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In this scenario, we need to ensure that any new service or update can't be deployed directly to production without meeting certain prerequisites. The governance model should include automated checks like rollback paths, health/version endpoints, authentication/authorization (authz) on every route, and the involvement of an owning on-call team. This is all enforced as part of a "production-readiness gate" rather than a manual checklist.

#### Triaging the Question

The core question here is about implementing robust CI/CD delivery governance to ensure that any service or change can't be released into production without meeting several critical requirements. The interviewer is probing how you would architect and enforce these checks as automated gates in your continuous deployment pipeline, ensuring high reliability and security.

#### Assumptions
We are working with a microservices architecture.
Each service has its own CI/CD pipeline.
Production environments include staging, pre-production, and production stages.
The on-call team is responsible for monitoring the service 24/7 and responding to incidents.
Authentication mechanisms (such as OAuth or JWT) are in place.

#### First Principles Solution

Rollback Path:

Implement a canary deployment strategy where a small percentage of traffic routes through the new version while most goes to the current production version.
If issues arise, roll back immediately by directing all traffic to the previous version.

Health and Version Endpoints:

Ensure each service has health checks available via /health endpoint that return the status of the microservice.
Include a versioning mechanism in the API responses or headers to track changes over time.

Authentication/Authorization (Authz):

Use an authentication service like Keycloak, Auth0, or Okta for managing user sessions and access tokens.
Implement fine-grained authorization using policies defined by RBAC (Role-Based Access Control) or ABAC (Attribute-Based Access Control).

Owning On-Call Team:

Define a clear ownership model where specific teams are responsible for monitoring and handling incidents.
Ensure that the on-call team has access to necessary logging, monitoring tools, and canary release controls.

Automated Production Readiness Gate:

Integrate these checks into your CI/CD pipeline as mandatory steps before promoting code to production.
Use a GitOps approach where changes are merged only after passing all automated tests and deployment validation steps.

#### Example
Architecture
Code Repository
CI System
CI Pipeline: Lint, Test, Build
Staging Environment
Canary Deployment Gateway
Pre-Prod Environment
Production Environment
Health/Version Endpoints
Authz Mechanisms
On-Call Monitoring System
AWS Route53 for DNSManagement

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, we've architected a robust CI/CD pipeline with automated gates to enforce production readiness. This includes canary deployments, health/version endpoints, authz mechanisms, and an on-call team. The key is ensuring that these checks are integrated into the CI/CD process as automated steps rather than relying on manual sign-offs.

#### Common Mistakes

Failing to automate all checks.
Overlooking the importance of rollback paths in critical services.
Not having clear ownership for monitoring and handling incidents.
Ignoring security aspects like authz mechanisms.

For follow-ups, we might discuss specific tools used (e.g., Prometheus for health checks, AWS CodePipeline for CI/CD), additional security measures, or how to scale this approach across multiple services.

**Comparison:**

| Stage | Service | Checks |
|-------|---------|--------|
| CI    | All     | Lint, Unit Tests, Integration Tests |
| CD1   | Staging | Health Checks, Version Check         |
| CD2   | Pre-Prod| Rollback Path, Canary Deployment     |
| Prod  | Production | N/A |

**Code:**

```yaml
# Example YAML for a GitOps deployment strategy

apiVersion: apps/v1
kind: Deployment
metadata:
  name: service-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: service
    spec:
      containers:
      - name: service-container
        image: my-service-image:v1.0.0
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: service-svc
spec:
  selector:
    app: service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

**Example input / output:**

```text
# Example Input -> Output for a Health Endpoint
Input:
- HTTP GET request to /health

Output:
{
  "status": "UP",
  "services": [
    {
      "name": "Database",
      "status": "OK"
    },
    {
      "name": "Cache",
      "status": "DEGRADED"
    }
  ]
}
```

**Data:**

```json
{
  "version": "1.0.0",
  "routes": [
    {
      "path": "/api/v1/",
      "methods": ["GET", "POST"],
      "authzPolicy": "admin-only"
    },
    {
      "path": "/health",
      "methods": ["GET"]
    }
  ]
}
```

**Flow:**

1. Code Commit
2. Ci System
3. Lint, Unit Tests, IntegrationTests
4. Staging Deploy
5. Health Check Pass?
6. Yes -> Pre-Prod Deploy
7. No -> Fix & Re-test
8. Canary Traffic Deploy
9. Monitor Canaries
10. All Good -> Prod Deploy
11. Rollback if Issues

