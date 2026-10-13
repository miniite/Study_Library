---
type: "practice"
difficulty: "Expert"
---

# How would you architect a cloud-native platform so that a single tenant's abusive traffic or expensive query cannot degrade the shared infrastructure, and how do you make fairness a first-class, measurable property?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Tenant Isolation](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Tenant_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a single tenant's abusive traffic or expensive queries don't degrade the shared cloud-native platform, we need to architect the system with robust isolation, cost management, and fair resource allocation mechanisms. The key is to treat fairness as a first-class property by designing the system in such a way that it measures, enforces, and reports on resource usage fairly across tenants.

#### Triaging the Question

The core of this question revolves around ensuring isolation and fairness within a cloud-native environment. Isolation means preventing one tenant's misbehavior (e.g., abusive traffic or expensive queries) from affecting others. Fairness involves allocating resources in a way that is perceived as equitable by all tenants, which can be measured using metrics like resource utilization and query cost.

#### Assumptions
Tenant: Each client or service consuming the platform.
Isolation Mechanisms: Techniques to limit the impact of one tenant's actions on others.
Cost Management: Methods for managing and billing tenants based on their resource consumption.
Fairness Metrics: Key performance indicators (KPIs) to measure fairness, such as resource usage percentages or query costs.

#### First-Principles Solution
Step 1: Define the Architecture

The architecture should be layered with clear boundaries between different components:

Network Layer: Ensures secure and isolated communication.
Compute Layer: Manages tenant workloads and allocates resources.
Storage Layer: Stores application data, ensuring isolation through multi-tenant storage solutions.
Cost Management Layer: Tracks resource usage per tenant.
Step 2: Implement Isolation
Network Segmentation: Use Virtual Private Clouds (VPC) to isolate traffic between tenants.
Resource Quotas: Set hard limits on compute and storage resources for each tenant.
Rate Limiting: Apply rate limiting at the network edge to prevent abusive traffic.
Step 3: Enforce Cost Management
Billing Policies: Define billing rules based on resource usage, ensuring fair cost allocation among tenants.
Cost Allocations: Use detailed cost reporting tools to track and bill resources consumed by each tenant.
Budgeting: Allow tenants to set spending limits and receive alerts when they approach or exceed their budgets.
Step 4: Measure Fairness
Usage Metrics: Collect metrics such as CPU, memory, storage, and network usage for each tenant.
Query Cost Metrics: Define a cost model based on query complexity and resource consumption to ensure fair billing.
Fairness Monitoring: Implement continuous monitoring and reporting mechanisms to enforce fairness KPIs.

#### Closing Summary

By combining isolation, cost management, and fairness metrics, we can build a robust cloud-native platform that ensures the shared infrastructure remains healthy and fair for all tenants. This involves careful design of network boundaries, resource quotas, billing policies, and monitoring tools.

#### Common Mistakes/Follow-Ups
Overlooking Isolation: Failing to properly isolate tenants can lead to cross-tenant attacks or overloading.
Simplistic Cost Models: Ignoring the complexity of different types of queries and their resource consumption may result in unfair billing practices.
Lack of Monitoring: Not implementing continuous monitoring tools for fairness and cost management can make it difficult to address issues promptly.

**Comparison:**

```
Tenant	CPU Usage %	Memory Usage %	Storage Usage %	Query Cost ($)
1	50	40	60	200
2	30	20	10	50
```

**Code:**

```python
# Example code for rate limiting in a cloud-native service

from flask import Flask, request
import time

app = Flask(__name__)

def rate_limit(limit):
    def decorator(f):
        cache = {}
        def wrapper(*args, **kwargs):
            key = str(request.remote_addr)
            if key not in cache:
                cache[key] = [time.time()]
            else:
                cache[key].append(time.time())
                # Remove old timestamps
                now = time.time()
                cache[key] = [t for t in cache[key] if t > (now - 60)]
                if len(cache[key]) > limit:
                    return "Rate Limit Exceeded", 429
            response = f(*args, **kwargs)
            return response
        return wrapper
    return decorator

@app.route('/api/query')
@rate_limit(10)  # Allow 10 requests per minute from each IP address
def query():
    # Handle the query logic here
    pass
```

**Example input / output:**

```text
Input:
Tenant 1 makes a series of queries that consume more CPU and memory than their quota.

Output:
- Tenant 1 is throttled to prevent overloading the system.
- A warning or alert is sent to the tenant's administrator to notify them of excessive usage.
```

**Data:**

```json
{
  "tenant_metrics": {
    "tenant1": {
      "cpu_usage": 50,
      "memory_usage": 40,
      "storage_usage": 60,
      "query_cost": 200
    },
    "tenant2": {
      "cpu_usage": 30,
      "memory_usage": 20,
      "storage_usage": 10,
      "query_cost": 50
    }
  },
  "fairness_monitoring": {
    "total_cpu_limit": 100,
    "total_memory_limit": 80,
    "total_storage_limit": 90,
    "budgeted_query_cost_per_tenant": [200, 50]
  }
}
```

**Flow:**

1. Yes
2. No
3. Tenant Request
4. Is Rate Limited?
5. Handle Request
6. Throttle Tenant & Alert Admin
7. Metric Collection
8. Fairness Monitoring

