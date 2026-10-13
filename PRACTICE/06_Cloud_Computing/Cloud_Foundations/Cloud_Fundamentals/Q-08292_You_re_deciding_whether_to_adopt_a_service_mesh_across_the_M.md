---
type: "practice"
difficulty: "Deep"
---

# You're deciding whether to adopt a service mesh across the ML serving fleet. Beyond the marketing, what specific cloud-networking, mTLS, and per-model routing problems does it actually solve, and what cheaper 80% alternatives would you try first?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Service Mesh](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Service_Mesh.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether to adopt a service mesh across an ML serving fleet, we need to look beyond marketing hype and focus on the specific problems it addresses in cloud networking, mTLS (mutual TLS), and per-model routing. A service mesh can significantly improve service discovery, traffic management, and security. However, there are cheaper alternatives that might suffice for 80% of use cases before fully committing.

#### Triaging the Question

The core problem we're addressing is enhancing the robustness, security, and scalability of our ML serving infrastructure. The key areas to focus on are:

Cloud Networking: Ensuring reliable and efficient traffic between services.
mTLS: Enhancing mutual authentication for secure communication.
Per-Model Routing: Handling different model versions or configurations.

#### First Principles Solution
Cloud Networking

A service mesh can address the challenges of managing complex microservices architectures by providing a unified way to control, monitor, and manage service-to-service (S2S) traffic. Key benefits include:

#### Cons
istent Behavior Across Services: Ensuring that all services follow consistent policies for retries, timeouts, circuit breakers, etc.
Traffic Management: Implementing advanced routing strategies like A/B testing or canary releases without modifying application code.
mTLS

mTLS is crucial for securing communication between services. In the context of ML serving fleets, this means:

Mutual Authentication: Both client and server must authenticate each other to ensure that only authorized parties communicate.
Data Encryption: Encrypting all traffic to protect sensitive data and prevent eavesdropping.
Per-Model Routing

Per-model routing is vital for efficiently managing different model versions or configurations. This can be achieved through:

Versioned APIs: Ensuring that clients can request specific model versions.
A/B Testing and Canaries: Gradually rolling out new models to a subset of traffic before full deployment.
Cheaper 80% Alternatives
For Cloud Networking

Kubernetes Service Mesh (Istio):

#### Pros
Provides robust service discovery, traffic management, and canary deployments.

#### Cons
Requires significant setup and ongoing maintenance. Istio is complex to configure and monitor.

Envoy Proxy:

#### Pros
A lightweight proxy that can be used independently or as part of a Kubernetes deployment. Offers flexible routing rules and health checks.

#### Cons
While simpler, it requires more manual configuration compared to full service mesh solutions like Istio.
For mTLS

Kubernetes TLS Secret Management:

#### Pros
Utilizes built-in Kubernetes mechanisms for managing TLS certificates and keys securely.

#### Cons
Limited flexibility in terms of mutual authentication and encryption options.

mTLS Using Mutual Authentication Libraries:

#### Pros
Implementing mutual authentication using libraries like mutual-tls or grpc-interceptors can be simpler to set up compared to complex service mesh solutions.

#### Cons
Requires more effort in handling certificate management and might not provide as robust security as a full service mesh.
For Per-Model Routing

Kubernetes Ingress Controller:

#### Pros
Provides a simple way to route traffic based on paths, headers, or cookies. Can be configured using annotations.

#### Cons
May require additional configuration for complex routing rules and versioning strategies.

Custom Service Discovery with Health Checks:

#### Pros
Implement custom solutions using Kubernetes services and health checks to dynamically route traffic to available model instances.

#### Cons
More manual effort in setting up discovery mechanisms and handling edge cases.

#### Closing Summary

While a full-fledged service mesh like Istio can provide comprehensive solutions for cloud networking, mTLS, and per-model routing, they come with significant overhead. For many use cases, simpler alternatives like Envoy Proxy, Kubernetes TLS secret management, or custom Ingress controllers can suffice. These options are easier to set up, maintain, and scale.

#### Common Mistakes
and Follow-Ups
Overcomplicating Solutions: Engineers might jump straight to complex service mesh solutions without considering simpler alternatives.
Neglecting Security: Focusing too much on performance while overlooking the importance of robust security measures like mutual TLS.
Ignoring Scalability: Underestimating the need for dynamic routing and traffic management, especially in large-scale ML serving fleets.

By thoroughly evaluating the specific requirements and carefully choosing between these options, we can build a more efficient and scalable infrastructure without over-engineering.

**Comparison:**

```
Solution	Benefits	Complexity	Cost
Istio Service Mesh	Consistent behavior across services, Advanced routing strategies	High	High
Envoy Proxy	Lightweight proxy with flexible rules, Easier to set up compared to full service mesh	Low	Medium
Kubernetes TLS Secrets Management	Built-in Kubernetes mechanisms for secure communication, Flexible key management	Low	Low
Custom Libraries for mTLS	Simplers implementation using mutual authentication libraries, Customizable security measures	High	Medium
Ingress Controller for Per-Model Routing	Simple path-based routing, Easily configured using annotations	Low	Low
Custom Health Checks and Discovery	Dynamic service discovery and health monitoring, Flexibility in handling model versions	High	Medium
```

**Code:**

```yaml
# Example Kubernetes Ingress Configuration for Per-Model Routing
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: model-ingress
spec:
  rules:
    - host: models.example.com
      http:
        paths:
          - path: /model-v1
            pathType: Prefix
            backend:
              service:
                name: model-service-v1
                port:
                  number: 80
```

**Example input / output:**

```text
# Example of Configuring Envoy Proxy for Per-Model Routing

Input:
- Model service A and B are deployed with different versions.
- We want to route traffic based on the path.

Output:
- Traffic is routed to model service A using /model-v1, and to model service B using /model-v2.

Request: GET http://models.example.com/model-v1
Response: Served by Model Service A

Request: GET http://models.example.com/model-v2
Response: Served by Model Service B
```

**Data:**

```json
{
  "serviceMesh": {
    "istio": {
      "benefits": [
        "Consistent behavior across services",
        "Advanced routing strategies"
      ],
      "complexity": true,
      "cost": "High"
    },
    "envoyProxy": {
      "benefits": [
        "Lightweight proxy with flexible rules",
        "Easier to set up compared to full service mesh"
      ],
      "complexity": false,
      "cost": "Medium"
    }
  },
  "mTLS": {
    "kubernetesSecrets": {
      "benefits": [
        "Built-in Kubernetes mechanisms for secure communication",
        "Flexible key management"
      ],
      "complexity": false,
      "cost": "Low"
    },
    "customLibraries": {
      "benefits": [
        "Simpler implementation using mutual authentication libraries",
        "Customizable security measures"
      ],
      "complexity": true,
      "cost": "Medium"
    }
  },
  "perModelRouting": {
    "ingressController": {
      "benefits": [
        "Simple path-based routing",
        "Easily configured using annotations"
      ],
      "complexity": false,
      "cost": "Low"
    },
    "customHealthChecks": {
      "benefits": [
        "Dynamic service discovery and health monitoring",
        "Flexibility in handling model versions"
      ],
      "complexity": true,
      "cost": "Medium"
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Service Mesh Decision
8. Cloud Networking Needs?
9. Kubernetes Service Mesh(Istio)
10. Envoy Proxy
11. mTLS Needs?
12. Mutual AuthenticationRequired?
13. Kubernetes TLS SecretsManagement
14. Custom Libraries for mTLS
15. Per-Model Routing Needs?
16. Dynamic Service DiscoveryNeeded?
17. Ingress Controller forPer-Model Routing
18. Custom Health Checks andDiscovery

