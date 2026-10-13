---
type: "practice"
difficulty: "Deep"
---

# Probe: you sized the platform for the analytics use-case, but now product wants it in the request path with 50ms SLAs — is that the same platform or a different one? How do you avoid one workload's needs distorting the whole strategy?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Open vs Proprietary Formats](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Open_vs_Proprietary_Formats.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, let's start by breaking down the core issue and key considerations before diving into how we can design a robust data platform strategy that meets both the analytics use-case and real-time request path requirements without distorting the overall platform strategy.

#### Triaging the Question

The question revolves around scaling a data platform to meet multiple workload requirements with distinct performance SLAs. Specifically, you initially sized the platform for an analytics use-case, which typically involves batch processing and can tolerate longer response times. Now, product management has requested real-time capabilities with strict 50ms SLAs in the request path.

Key Considerations
Separate Workloads: The primary challenge is whether these workloads can coexist on a single platform without one set of requirements dominating the others.
Resource Allocation and Isolation: Ensuring that resources are appropriately allocated to meet both sets of SLAs.
Platform Design Principles: Consideration of architectural patterns, such as microservices or serverless functions, which might help in isolating workloads.
Cost Efficiency: Balancing the cost of maintaining a single platform versus potentially deploying multiple platforms for different use-cases.

#### First-Principles Solution

Assess Current Platform Capabilities:

Evaluate the current architecture and resource utilization under the analytics workload.
Identify bottlenecks and performance constraints that might not be suitable for real-time processing.

Determine Feasibility of Coexistence:

If the existing platform can handle both workloads, refactor it to ensure isolation and efficient resource management.

#### Cons
ider implementing a multi-tenancy solution or container orchestration (e.g., Kubernetes) to isolate different types of workloads while sharing common resources.

Design for Scalability and Isolation:

Utilize horizontal scaling techniques such as auto-scaling groups in AWS, Kubernetes Pods, or Azure Service Fabric.
Implement caching mechanisms like Redis or Memcached to reduce latency.
Use queuing systems (e.g., Apache Kafka) for decoupling different workloads.

Architectural Patterns:

Serverless Functions: Leverage serverless computing services that automatically scale based on demand, reducing the need for manual resource management.
Microservices Architecture: Design microservices to handle specific real-time processing tasks with their own SLAs, ensuring they do not interfere with batch analytics workloads.

Performance Monitoring and Optimization:

Implement comprehensive monitoring tools (e.g., Prometheus, Grafana) to continuously monitor performance metrics.
Use A/B testing to validate the effectiveness of different architectural decisions without impacting production systems.

#### Closing Summary

In summary, while it might be feasible to coexist with both analytics and real-time request path workloads on a single platform, careful design and implementation are necessary. By leveraging modern cloud-native technologies like Kubernetes, serverless functions, and robust monitoring tools, you can ensure that the platform meets all required SLAs without being distorted by one workload's needs.

#### Common Mistakes/Follow-Ups
Overlooking Resource Contention: Failing to properly manage resource allocation could lead to performance degradation for both workloads.
Neglecting Isolation Mechanisms: Not implementing proper isolation can cause real-time requests to block analytics processing or vice versa.
Underestimating Cost Implications: The cost of maintaining a hybrid platform might be higher than deploying separate platforms, especially if the current platform cannot be refactored effectively.

For deeper discussions, we could explore specific use-case examples and perform dry-runs using sample data to illustrate potential bottlenecks and optimization strategies.

**Comparison:**

| Resource Type | Limit | Request |
|---------------|-------|---------|
| CPU           | 1     | 0.5     |
| Memory        | 512Mi | 256Mi   |

**Code:**

```python
# Example of scaling logic in a Kubernetes deployment YAML file
apiVersion: apps/v1
kind: Deployment
metadata:
  name: real-time-processing-deployment
spec:
  replicas: 3 # Initial number of pods, can be dynamically adjusted based on traffic patterns
  selector:
    matchLabels:
      app: real-time-processing
  template:
    metadata:
      labels:
        app: real-time-processing
    spec:
      containers:
      - name: real-time-processing-container
        image: <real-time-processing-image>
        resources:
          limits:
            cpu: "1"
            memory: "512Mi"
          requests:
            cpu: "0.5"
            memory: "256Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: real-time-processing-service
spec:
  selector:
    app: real-time-processing
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

**Example input / output:**

```text
# Example of a dry-run input-output scenario for a serverless function

Input (Request):
{
  "user_id": "123",
  "timestamp": "2023-10-04T10:00:00Z"
}

Output (Response):
{
  "status": "success",
  "data": {
    "processed_data": [
      {"key": "value"},
      ...
    ]
  }
}
```

**Data:**

```yaml
# Example of a Kubernetes ConfigMap for environment variables
apiVersion: v1
kind: ConfigMap
metadata:
  name: real-time-processing-config
data:
  REAL_TIME_PROCESSING_ENV_VAR: "production"
---
apiVersion: v1
kind: Secret
metadata:
  name: real-time-processing-secret
type: Opaque
data:
  API_KEY: <encoded_api_key>
```

**Flow:**

1. Yes
2. No
3. Real-Time Processing
4. Isolation Mechanisms in Place?
5. Monitor Performance Metrics
6. Leverage Kubernetes forIsolation
7. Maintain SLAs
8. Implement ServerlessFunctions
9. Optimize Cost andPerformance

