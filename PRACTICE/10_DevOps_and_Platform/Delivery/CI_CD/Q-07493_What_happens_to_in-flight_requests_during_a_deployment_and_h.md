---
type: "practice"
difficulty: "Deep"
---

# What happens to in-flight requests during a deployment and how do you avoid dropping them?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Deployment Observability](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this deep dive into CI/CD, we'll explore what happens to ongoing requests when a backend service is deployed, and discuss strategies to ensure these in-flight requests aren't dropped during the transition period. The core challenge lies in maintaining service continuity while performing updates without downtime or data loss.

The question probes your understanding of the interaction between live user traffic and continuous deployment pipelines, as well as your ability to implement robust solutions that handle unexpected issues gracefully. Key assumptions include:

You are using a typical CI/CD pipeline with automated deployments.
The backend service is stateful but can tolerate short periods of unavailability.

Let's break this down step by step from first principles:

Understanding the Problem: When a deployment occurs, any in-flight requests during that period could be dropped if the server stops responding or crashes before fully serving all those requests. This includes both read and write operations, which can lead to data inconsistency issues.

Strategies to Avoid Dropped Requests:

Blue-Green Deployment: Maintain two identical environments (blue and green) and switch traffic seamlessly between them.
Canary Releases: Gradually introduce the new version of your application by directing a small percentage of traffic to it, allowing for gradual rollouts and monitoring before full-scale deployment.

Detailed Implementation:

Blue-Green Deployment Example:

Initial Blue Environment
New Green Environment
Final Switch Traffic from Blueto Green
Both Environments Running
Remove Old Blue Environmentif No Issues Detected

Canary Release Example:

Initial Environment
Deploy New Version to aSubset of Users (10%)
Monitor Performance and UserFeedback
Gradually IncreaseDeployment Percentage ifSuccessful
Full Rollout if Everything LooksGood

Technical Considerations:

Load Balancers: Use load balancers to route traffic dynamically between different environments.
Caching and Session Management: Ensure that session management is handled correctly, especially in stateful applications where user sessions need to be preserved across deployments.

#### Closing Summary
and Common Mistakes/Follow-Ups:

Ensuring service availability during deployment is crucial for maintaining user trust and avoiding potential data loss.
Failing to plan for edge cases like unexpected traffic spikes or system crashes can lead to dropped requests and other issues.
It's important to test the entire CI/CD pipeline thoroughly, including rollback strategies in case of failures.

**Comparison:**

| Deployment Strategy | Description | Key Advantages | Considerations |
|---------------------|-------------|----------------|---------------|
| Blue-Green          | Two identical environments, traffic switch is seamless.| Minimal downtime risk, easy rollback. | Requires double the resources. |
| Canary Release      | Gradual rollout of new version with minimal impact. | Smooth transition, low risk of issues. | Monitoring and gradual increase needed. |

**Code:**

```python
# Example of a simple blue-green deployment switch using Flask and Gunicorn
import os

def get_current_environment():
    return os.getenv('ENVIRONMENT', 'blue')

def route_request(request):
    env = get_current_environment()
    if env == 'green':
        # Serve requests from the green environment
        pass
    else:
        # Serve requests from the blue environment
        pass
```

**Example input / output:**

| Input | Output |
|-------|--------|
| User request to `/api/v1/resource` during blue-green deployment | Request is routed based on load balancer configuration and current environment setting. If the request hits green backend, it gets processed there; otherwise, it processes through blue backend. |

**Data:**

```yaml
# Sample YAML configuration for a load balancer to route traffic between environments
load_balancer_config:
  - name: blue-backend
    port: 8080
  - name: green-backend
    port: 8081
  routing_policy:
    strategy: round-robin
```

**Flow:**

1. Initial Environment
2. Deploy New Version to aSubset (10%)
3. Monitor Performance and UserFeedback
4. Gradually IncreaseDeployment Percentage ifSuccessful
5. Full Rollout if Everything LooksGood

