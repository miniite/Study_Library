---
type: "practice"
difficulty: "Expert"
---

# A deploy needs to atomically flip both a service version and its downstream service-mesh routing weights; how do you architect the coordination so the mesh and the CD system never disagree about the intended state?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Database Migrations in Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Database_Migrations_in_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure atomic flips of both a service version and its downstream service-mesh routing weights, we need to design a robust CI/CD pipeline that coordinates these changes atomically. This is crucial to avoid a window where the mesh configuration does not match the intended state, leading to potential service outages or degraded performance.

#### Triaging the Question

The core of this question revolves around ensuring coordinated updates between your application's deployment and its routing weights in a service mesh. The interviewer is probing on how you handle concurrency, coordination, and rollback mechanisms to prevent partial deployments or inconsistencies.

#### Assumptions
and Key Considerations
Service Mesh: We assume the use of a modern service mesh like Istio or Linkerd for managing service-to-service communication.
Deployment Strategy: The deployment strategy involves rolling updates where new versions are introduced gradually, replacing old ones over time.
Routing Weights: Service mesh routing weights control how traffic is directed to different service instances (old vs. new).

#### First-Principles Solution
Versioning and Tagging:
Use semantic versioning for your services to ensure clear identification of versions.
Coordination Mechanism:
Implement a coordination mechanism that ensures both the deployment change and the corresponding routing weight update are handled as a single atomic operation.
Rollout Strategy:
Gradually increase traffic to new service instances while decreasing it from old ones, ensuring smooth transition.
Step-by-Step Solution
Version Tagging:
Label your services with version tags (e.g., v1, v2).
Preparation Phase:
Update the deployment pipeline to tag new service instances with the next version label.
Routing Weight Adjustment:
Simultaneously, update the service mesh's routing rules to reflect these changes. This can be done using a custom operator or by updating the mesh configuration via API.
Rollout Execution:
Gradually increase traffic to new instances while decreasing it from old ones. Ensure this is done in lockstep with the version tagging and routing weight adjustments.

#### Example Scenario

#### Cons
ider a service that needs an atomic flip from v1 to v2. The steps would be:

Deploy v2 instances.
Update routing rules to route 50% of traffic to v2.
Gradually increase traffic to v2 while decreasing it for v1, ensuring the transition is smooth.

#### Closing Summary

By coordinating the deployment and service mesh routing weight updates atomically, we ensure that there are no windows where the intended state (mesh configuration) does not match the actual deployed state. This approach provides a robust solution to avoid partial deployments or service outages.

#### Common Mistakes/Follow-Ups
Partial Deployment: Failing to ensure both deployment and mesh routing updates are handled as an atomic operation.
Rollback Mechanism: Not having a solid rollback plan in case of issues during the transition.
Monitoring and Alerting: Overlooking critical monitoring and alerting to detect discrepancies early.

**Code:**

```yaml
# Example Istio Configuration for Rolling Update
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: service-mesh-routing-rule
spec:
  host: my-service
  subsets:
    - name: v1
      labels:
        version: "v1"
    - name: v2
      labels:
        version: "v2"

---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: service-mesh-traffic-split
spec:
  hosts:
    - my-service
  gateways:
    - istio-system/gateway
  http:
  - match:
    - headers:
        version:
          exact: v2
    route:
    - destination:
        host: my-service
        subset: v2
  - route:
    - destination:
        host: my-service
        subset: v1
```

**Flow:**

1. Prepare Deployment
2. Update Mesh Routing Rules
3. Rollout Traffic to New Version
4. Monitor and Ensure SmoothTransition
5. Rollback Plan in Place

