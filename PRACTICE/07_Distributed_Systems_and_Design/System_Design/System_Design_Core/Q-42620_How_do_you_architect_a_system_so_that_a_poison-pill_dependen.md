---
type: "practice"
difficulty: "Expert"
---

# How do you architect a system so that a poison-pill dependency (a widely-used internal library with a critical bug) can be patched and rolled out across 300 services in hours, not weeks? What platform investment makes that possible?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Coordinated Client Upgrades](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Coordinated_Client_Upgrades.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a system where a critical poison-pill dependency can be patched and rolled out to hundreds of services quickly, we need to focus on modularity, decoupling, and resilient deployment strategies. The key is to ensure that each service is as independent as possible from the problematic library while still being able to seamlessly transition to a new version when available.

First, let's break down what the question is really asking: the goal is to design a system where individual services can be decoupled from critical dependencies, and updates to these dependencies can be performed in a controlled manner without causing downtime or significant delays. The interviewer will probe into your understanding of microservices architecture, deployment pipelines, and resilience strategies.

#### Triage

The core challenge here is ensuring that even if the library contains a critical bug, it doesn't bring down the entire system. We need to implement a strategy where services can continue operating normally while updates are being rolled out in parallel or sequentially, depending on the specific requirements.

#### Assumptions
Each service uses the problematic library for some of its operations.
The new version of the library is available and ready for deployment.
There is an automated and reliable way to deploy changes across services.

#### Solution

The first step is decoupling the services from the critical dependency. This involves several layers:

Service-Driven Design: Ensure each service has minimal direct dependencies on external libraries. Instead, use well-defined interfaces or APIs for communication.
Dependency Injection: Use a framework that allows for easy swapping of implementations. For example, use a library like Spring in Java where beans can be easily configured to point to different implementations.
API Gateway: Implement an API gateway (e.g., NGINX, Kong) that acts as a proxy between external services and your microservices. This way, you can control the flow of requests and manage dependencies at one central point.

To handle the rollout:

A/B Testing: Gradually introduce the new version to a subset of users/services to test for any issues.
Blue/Green Deployments: Maintain two identical environments (blue and green) where changes are deployed to one environment first, tested, and then switched over once everything is stable.
Canary Releases: Start by deploying the new library to a small percentage of services and monitor their performance before rolling out fully.

Finally, ensure you have robust logging, monitoring, and alerting mechanisms in place so that any issues can be detected and addressed quickly.

#### Closing Summary

The key to achieving this is a combination of well-designed microservices, automated deployment pipelines, and resilient strategies like A/B testing and blue/green deployments. The platform investments required include adopting modern container orchestration tools (like Kubernetes), implementing robust CI/CD pipelines, and setting up comprehensive monitoring and alerting systems.

#### Common Mistakes

Failing to decouple services from the problematic library.
Not having a strategy for gradual rollout and testing.
Underestimating the importance of robust logging and monitoring.
Closing Note

By investing in these areas, you can minimize downtime and ensure that critical updates can be managed efficiently without impacting the entire system. This not only ensures business continuity but also allows for continuous improvement and innovation.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Service] --> B[External Dependency];
    C[API Gateway] --> D[Blue Environment];
    E[Green Environment] --> F[Production];
    G[Canary Release] --> H[Test Users];
```

