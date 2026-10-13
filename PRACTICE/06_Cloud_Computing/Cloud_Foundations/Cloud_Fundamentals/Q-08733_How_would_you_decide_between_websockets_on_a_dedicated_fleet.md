---
type: "practice"
difficulty: "Deep"
---

# How would you decide between websockets on a dedicated fleet versus a managed real-time service for a chat feature expecting 100k concurrent connections?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Sync vs Async Processing](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Sync_vs_Async_Processing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide between using websockets on a dedicated fleet versus a managed real-time service for a chat feature expecting 100k concurrent connections, we need to carefully weigh several factors including scalability, reliability, cost, and ease of deployment. We'll start by understanding the core concepts and then proceed step-by-step from first principles.

First, let's define the key terms:

Websockets on a Dedicated Fleet: This involves setting up and maintaining your own infrastructure for handling websockets, which includes managing servers, networking, security, scaling, and potentially dealing with failures.
Managed Real-Time Service: These are cloud-provided services that abstract away much of the complexity of setting up and managing real-time communication. Examples include AWS AppSync, Firebase Realtime Database, or Google Cloud Pub/Sub.

The question is essentially probing our ability to evaluate different technical strategies based on specific use cases, understanding their trade-offs, and making informed decisions.

Step-by-Step Analysis

Scalability:

Websockets on a Dedicated Fleet: This approach offers full control over scaling, allowing for dynamic adjustments based on real-time load data. However, it requires significant expertise in managing infrastructure.
Managed Real-Time Service: These services are typically designed to automatically scale based on demand without much user intervention.

Reliability:

Websockets on a Dedicated Fleet: Implementing robust error handling and redundancy strategies can be complex but is within the realm of what experienced teams can manage.
Managed Real-Time Service: Cloud providers generally offer high reliability, including built-in failover mechanisms and automatic backups.

Cost:

Websockets on a Dedicated Fleet: Initial setup costs for hardware and software licensing could be high. Ongoing maintenance, scaling, and potential spikes in demand can lead to unpredictable costs.
Managed Real-Time Service: Usually offers a predictable cost model based on usage, which simplifies budgeting.

Ease of Deployment and Maintenance:

Websockets on a Dedicated Fleet: Requires more effort in setting up the infrastructure and maintaining it, including regular updates and security patches.
Managed Real-Time Service: Simplifies deployment as much of the heavy lifting is done by the provider, reducing maintenance burden.

Latency and Performance:

Both options can be optimized for low latency if appropriate network configurations are in place. However, cloud services might have inherent delays due to data transmission over public networks.

Security Considerations:

Websockets on a Dedicated Fleet: Requires careful configuration of firewalls, SSL/TLS certificates, and other security measures.
Managed Real-Time Service: Often includes built-in security features but still requires adherence to best practices for secure usage.

#### Conclusion

For an application with 100k concurrent connections, the managed real-time service is often a better choice due to its lower operational overhead, predictable cost structure, and robust reliability. However, if there are specific requirements that necessitate full control over infrastructure or if the team has significant expertise in managing such systems, then setting up websockets on a dedicated fleet might be justified.

In practice, we would likely conduct a pilot project using both approaches to compare performance and operational costs before making a final decision.

#### Common Mistakes/Follow-Ups
Overlooking Cost Analysis: Failing to consider the long-term financial implications of each option.
Neglecting Scalability Needs: Assuming managed services always scale better without considering specific application requirements.
Underestimating Expertise Required: Assuming that a team with less experience can handle complex infrastructure management.

**Comparison:**

```
Factor	Websockets on Dedicated Fleet	Managed Real-Time Service
Scalability	High with manual adjustments	Auto-scaling
Reliability	Requires robust error handling	Built-in failover and backups
Cost	Initial setup + ongoing maintenance	Predictable cost model
Ease of Use	Higher initial complexity	Lower operational overhead
Latency	Can be optimized	Varies based on network latency
```

**Code:**

```python
# Example Python code for a basic websocket setup (simplified)
import asyncio
from websockets.server import serve

async def handler(websocket, path):
    async for message in websocket:
        print(f"Received {message}")

asyncio.get_event_loop().run_until_complete(
    serve(handler, "localhost", 8765)
)
asyncio.run(serve.wait_closed())
```

**Example input / output:**

```text
Input:
- Number of connections = 100,000
- Expected concurrent users = 20,000

Output:
For managed service:
- Cost: $50 per month (assuming 2% data transfer rate)
- Reliability: High with built-in failover mechanisms

For dedicated fleet:
- Initial cost: $50k setup + $1000/month
- Scalability: Full control over scaling and configuration
```

**Data:**

```json
{
  "connections": 100000,
  "expected_concurrent_users": 20000,
  "cost_model": {
    "managed_service": "$0.5 per GB of data transfer",
    "dedicated_fleet": "Initial setup $50k + $1000/month maintenance"
  }
}
```

**Flow:**

1. Define requirements
2. Scalability needs
3. Reliability considerations
4. Cost analysis
5. Ease of use
6. Latency and performance
7. Security concerns
8. Choose between managed ordedicated

