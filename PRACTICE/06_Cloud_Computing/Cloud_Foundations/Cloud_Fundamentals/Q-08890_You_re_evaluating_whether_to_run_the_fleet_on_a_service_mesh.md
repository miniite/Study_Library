---
type: "practice"
difficulty: "Expert"
---

# You're evaluating whether to run the fleet on a service mesh company-wide. Beyond the marketing, what specific cloud-networking problems does a mesh solve, and what cheaper 80%-solutions would you try first?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Service Mesh](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Service_Mesh.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In evaluating whether to adopt a service mesh on a fullstack company-wide basis, it's crucial to understand that a service mesh primarily addresses cloud-networking challenges related to application-level communications, service discovery, and traffic management. Specifically, it simplifies the management of microservices, enhances observability, and improves security by handling these concerns at the infrastructure level rather than within each service.

The key problems a service mesh solves include:

Service Discovery: Ensures that services can reliably find and communicate with one another in complex, dynamic cloud environments.
Traffic Management: Provides control over how requests are routed between services, including features like retries, timeouts, and circuit breaking.
Observability: Offers comprehensive logging, tracing, and metrics to monitor and troubleshoot issues.
Security: Handles secure communication (mTLS), rate limiting, and authentication.

However, before committing to a full service mesh implementation, it’s worth exploring more cost-effective alternatives that can provide 80% of the benefits:

Sidecar Proxy Pattern: Deploying Envoy or NGINX as sidecars with each microservice can offer many of the same functionalities without the overhead of a dedicated service mesh.
Network Policies and Load Balancers: Utilizing Kubernetes network policies to control traffic between pods and using load balancers for basic routing and failover can cover some aspects of service discovery and traffic management.
API Gateway Solutions: An API gateway can handle many of the networking challenges, including rate limiting, authentication, and basic load balancing.

Let's break down each approach:

#### Triaging the Question

The question asks about evaluating a service mesh versus cheaper alternatives. The interviewer is probing your understanding of the core problems that a service mesh solves and your ability to propose practical, cost-effective solutions for those same issues. Key assumptions include the current state of the company’s infrastructure (e.g., Kubernetes deployment) and its microservices architecture.

#### First-Principles Solution
1. Service Discovery
Full Service Mesh: Implement a service mesh like Istio or Linkerd to manage service discovery dynamically.
Sidecar Proxy Pattern: Use Envoy or NGINX as sidecars with each microservice for basic service discovery and routing.
2. Traffic Management
Full Service Mesh: Utilize the advanced traffic management features provided by a service mesh, such as retries, timeouts, circuit breaking, and advanced load balancing policies.
Network Policies and Load Balancers: Configure Kubernetes network policies and use built-in load balancers for basic routing and failover.
3. Observability
Full Service Mesh: Leverage the built-in observability features of a service mesh, including comprehensive logging, tracing, and metrics.
Sidecar Proxy Pattern or API Gateway Solutions: Use sidecars like Envoy with logging and monitoring tools (e.g., Prometheus) or an API gateway for observability.
4. Security
Full Service Mesh: Implement mTLS, rate limiting, and authentication at the mesh level.
Sidecar Proxy Pattern or Network Policies: Deploy security policies and use built-in features of Kubernetes to enforce security best practices.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while a full service mesh offers robust solutions for all these problems, sidecar proxies and API gateways can provide 80% of the benefits at a lower cost. However, it's essential to consider the trade-offs in terms of complexity, supportability, and potential vendor lock-in.

#### Common mistakes
include underestimating the operational overhead of a service mesh or dismissing its benefits too quickly without fully evaluating the alternatives.

**Comparison:**

```
Service Mesh Feature	Sidecar Proxy Pattern / API Gateway Solutions
Service Discovery	Network Policies, Envoy sidecars
Traffic Management	Basic Load Balancers, Kubernetes policies
Observability	Prometheus/Monitoring Tools
Security	Kubernetes network policies, mTLS support
```

**Code:**

```
Example configuration for Kubernetes network policies

apiVersion: networking.k8s.io/v1 kind: NetworkPolicy metadata: name: allow-internal-tcp spec: podSelector: matchLabels: app: backend policyTypes:

Ingress ingress:
from:
podSelector: matchLabels: app: frontend ports:
protocol: TCP port: 80
CODE
Copy

```yaml
# Example Envoy sidecar configuration snippet
static_resources:
  clusters:
  - name: example_cluster
    connect_timeout: 10s
    type: logical_dns
    load_assignment:
      cluster_name: example_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: example-service
                port_value: 8080
```

**Example input / output:**

```text
# Example input and output for network policies
Input (NetworkPolicy YAML):
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-internal-tcp
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80

Output (NetworkPolicy applied):
Pods labeled 'app: backend' can only receive traffic from pods labeled 'app: frontend' on port 80.
```

**Data:**

```yaml
# Kubernetes Network Policy YAML snippet
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-internal-tcp
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
```

**Flow:**

1. Service Mesh
2. Full Service Discovery
3. Advanced Traffic Management
4. Comprehensive Observability
5. Security at the Mesh Level
6. Sidecar Proxy Pattern
7. Basic Service Discovery viaSidecars
8. Basic Load Balancing &Policies
9. Prometheus for Monitoring
10. mTLS Support for Security
11. Network Policies
12. Kubernetes Load Balancers
13. Prometheus/Tracing
14. mTLS, Rate Limiting

