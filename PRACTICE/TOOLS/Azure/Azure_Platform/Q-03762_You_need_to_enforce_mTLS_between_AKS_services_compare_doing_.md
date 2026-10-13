---
type: "practice"
difficulty: "Expert"
---

# You need to enforce mTLS between AKS services; compare doing it with a service mesh (Istio/Linkerd/Open Service Mesh) versus app-level, and the operational cost.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ AKS and Container Platform Standards](<../../../../TOOLS/Azure/Azure_Platform/Azure_Functions_and_Cosmos_DB_Tuning/AKS_and_Container_Platform_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce mutual TLS (mTLS) between Kubernetes services in an Azure environment, we need to consider both a service mesh approach using Istio/Linkerd/Open Service Mesh and an app-level solution. The choice depends on factors like complexity, operational overhead, and the desired level of control over network policies.

#### Triaging the Question

The core question here is comparing two different approaches for enforcing mTLS in Azure Kubernetes Services (AKS). We need to evaluate:

Service Mesh Approach: Utilizing Istio/Linkerd/Open Service Mesh.
App-Level Approach: Implementing mTLS at the application level.

#### Assumptions
Both AKS and service mesh are running on a cloud environment, likely Azure Cloud.
The services communicate over a network that can be potentially insecure.

#### First-Principles Solution
Service Mesh Approach (Istio/Linkerd/Open Service Mesh)
Network Policy Enforcement: A service mesh intercepts all inter-service communication and enforces mTLS by default. This means no modification is needed in the application code.
Operational Complexity:

#### Pros
Centralized policy management, automatic sidecar injection, easier to maintain and scale.

#### Cons
Additional components to deploy and manage (sidecars), potential overhead on performance due to interception.
Security Features:
Built-in features like rate limiting, authentication, and monitoring enhance security beyond just mTLS.
Scalability:
Service mesh can handle high volumes of traffic more efficiently compared to app-level solutions.
App-Level Approach
Network Policy Enforcement: Requires each application to be updated to enable TLS termination and mutual authentication.
Operational Complexity:

#### Pros
No additional components, simpler deployment.

#### Cons
Each application needs to implement mTLS, which can be error-prone and complex.
Security Features:
Limited to implementing mTLS; no built-in features for additional security measures.
Scalability:
Can become cumbersome with a large number of services.

#### Closing Summary

For enforcing mTLS between AKS services, using a service mesh like Istio/Linkerd/Open Service Mesh offers significant advantages in terms of operational simplicity and enhanced security beyond just TLS. However, it comes with the trade-off of increased complexity due to additional components. On the other hand, implementing app-level mTLS is simpler but less secure and more error-prone.

#### Common Mistakes/Follow-Ups
Overlooking Sidecar Injection: Service mesh solutions rely on sidecars for interception; forgetting this can lead to service failures.
Ignoring Security Best Practices: While a service mesh provides additional security features, it's still important to follow best practices like regular certificate rotations and key management.
Not Considering Performance Impact: The overhead of mTLS enforcement by a service mesh needs careful consideration, especially in high-throughput environments.

**Comparison:**

| Feature          | Service Mesh (Istio) | App-Level |
|------------------|----------------------|-----------|
| Complexity       | High                 | Low       |
| Security Features| Rate limiting, auth  | Basic     |
| Scalability      | Good                 | Poor      |
| Maintenance      | Centralized          | Distributed |

**Code:**

```yaml
# Example Istio configuration for mutual TLS
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
spec:
  profile: default
  components:
    pilot:
      enabled: true
    citadel:
      enabled: false # Use a custom CA
    kiali:
      enabled: true
```

**Example input / output:**

```text
Input:
- Service A (serviceA) in namespace default needs to communicate with Service B (serviceB) in namespace test.

Output:
1. **Service Mesh Approach**:
   - Deploy Istio with sidecar injection for both services.
   - Configure mutual TLS at the mesh level using a custom CA.
   - Services will automatically establish secure connections via mTLS.

2. **App-Level Approach**:
   - Update Service A and B to include TLS termination logic.
   - Ensure each service has a valid TLS certificate issued by an appropriate CA.
```

**Data:**

```json
{
  "services": [
    {
      "name": "serviceA",
      "namespace": "default"
    },
    {
      "name": "serviceB",
      "namespace": "test"
    }
  ]
}
```

**Flow:**

1. Deploy Istio with SidecarInjection
2. Intercept All Traffic
3. Configure Mutual TLS
4. Services CommunicateSecurely
5. Update Applications
6. TLS Configuration
7. Secure Communication

