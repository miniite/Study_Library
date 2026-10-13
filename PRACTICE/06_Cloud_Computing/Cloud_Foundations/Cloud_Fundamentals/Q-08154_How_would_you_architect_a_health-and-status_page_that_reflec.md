---
type: "practice"
difficulty: "Intermediate"
---

# How would you architect a health-and-status page that reflects real dependency health so customers see accurate incident status without you manually updating it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Health Checks and Load Balancers](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Health_Checks_and_Load_Balancers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

The key to building an automated health-and-status page lies in creating a microservices-based architecture where each service manages its own health and reports it to a central aggregator. This setup ensures that customers always see real-time status without manual intervention.

#### Triaging the Question

This question is asking you to design an automated system for displaying the overall health of multiple dependencies, such as databases, APIs, and external services, in a way that reflects their actual operational state rather than relying on manual updates. The core challenge here involves ensuring data consistency and real-time accuracy without human intervention.

#### First-Principles Solution

Microservices Architecture:

Each service should have its own health endpoint (e.g., /health) to report its status.
These endpoints can use standard protocols like HTTP/JSON or gRPC.

Central Aggregator Service:

A central service aggregates the health statuses from all dependencies and presents a unified view.
This aggregator should be highly available, scalable, and resilient to ensure no single point of failure impacts the status page.

Health Reporting Mechanism:

Implement periodic checks (e.g., every 5 minutes) by the aggregator service.
Use distributed tracing tools like Jaeger or Zipkin for monitoring the health-check process.

Failure Detection and Recovery:

Define clear failure criteria for each dependency.
Implement retries, circuit breakers, and fallback strategies to handle transient failures gracefully.

Monitoring and Alerting:

Set up alerting mechanisms based on predefined thresholds (e.g., critical errors, high latency).
Integrate with monitoring tools like Prometheus or Grafana for real-time visibility into system health.

Caching and Latency Considerations:

Use caching strategies to minimize the impact of slow dependency services.
Implement load balancing to distribute requests evenly among available dependencies.

#### Closing Summary

The solution involves a microservices architecture where each service manages its own health, which is then aggregated by a central service. This approach ensures real-time and accurate status updates without manual intervention. Key considerations include reliability, scalability, and responsiveness, especially in the presence of failures or high latency issues.

#### Common Mistakes/Follow-Ups
Manual Updates: Relying on human intervention to update statuses can lead to inaccuracies.
Single Point of Failure: Over-reliance on a single aggregator service can create a bottleneck.
Latency and Performance: Poor performance in individual services can affect the overall health reporting, leading to false positives or negatives.

**Comparison:**

```
Dependency	Health Endpoint	Status
db-service	http://db-service/health	True
api-service	http://api-service/health	False
```

**Code:**

```python
# Example of a simple health check endpoint for a microservice

from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/health')
def health_check():
    status = {'status': 'ok'}
    return jsonify(status)

if __name__ == '__main__':
    app.run(port=5000)
```

**Example input / output:**

```text
Input:
POST /check-dependency-status HTTP/1.1
Host: status-aggregator.example.com

{
  "dependencies": [
    {
      "id": "db-service",
      "healthUrl": "http://db-service/health"
    },
    {
      "id": "api-service",
      "healthUrl": "http://api-service/health"
    }
  ]
}

Output:
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": [
    { "id": "db-service", "health": true },
    { "id": "api-service", "health": false }
  ]
}
```

**Data:**

```json
{
  "dependencies": [
    {
      "id": "db-service",
      "healthUrl": "http://db-service/health"
    },
    {
      "id": "api-service",
      "healthUrl": "http://api-service/health"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Dependency Service
4. IsHealthy?
5. Report Status
6. Try Again Later
7. Central Aggregator
8. Aggregate Status
9. Send to UI

