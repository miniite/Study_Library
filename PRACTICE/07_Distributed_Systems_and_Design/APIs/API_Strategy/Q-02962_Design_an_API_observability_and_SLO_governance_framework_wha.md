---
type: "practice"
difficulty: "Expert"
---

# Design an API observability and SLO governance framework: what SLIs do you standardize per endpoint (availability, latency, correctness), how do error budgets gate deployment, and how do you attribute an SLO breach to the right owning team in a call chain?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Org-wide API Standards](<../../../../07_Distributed_Systems_and_Design/APIs/API_Strategy/Org-wide_API_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're tasked with designing a robust API observability and Service Level Objective (SLO) governance framework that ensures high availability, low latency, and correctness of endpoints, while effectively managing error budgets and attributing SLO breaches to the appropriate teams. The key is to establish clear, standardized SLIs for each endpoint, implement an error budget mechanism that gates deployments based on performance, and a robust tracing system that pinpoints responsible teams in case of SLO violations.

The question essentially probes our understanding of API management best practices, error handling strategies, and effective monitoring techniques. We will assume the following:

The API is part of a distributed system with multiple endpoints.
Each endpoint has unique SLIs defined based on its role.
Error budgets are used to control deployment risk.

Let's break down the solution from first principles:

Step 1: Defining SLIs per Endpoint

For each API endpoint, we'll define three key SLIs:

Availability: The percentage of requests that succeed without errors (e.g., HTTP 200 or 4xx/5xx status codes).
Latency: The average time a request takes to be processed and returned.
Correctness: The rate at which responses meet business requirements, often measured as the proportion of requests where the response is correct according to predefined criteria.
Step 2: Implementing Error Budgets

Error budgets are critical for controlling deployment risk. We'll set a fixed budget per endpoint that represents the maximum allowable percentage of requests that can fail without causing an SLO breach:

Determine Initial Error Budgets: Allocate initial error budgets based on historical data and SLI targets.
Monitor and Adjust Error Budgets: Continuously monitor error rates and adjust budgets as needed to maintain a balance between innovation and reliability.
Step 3: Attributing SLO Breaches

To attribute SLO breaches effectively, we need a comprehensive tracing system:

Implement Distributed Tracing: Use tools like Jaeger or OpenTelemetry to trace request flows across the API call chain.
Integrate with Monitoring Systems: Connect tracing data to monitoring and alerting systems to quickly identify the root cause of SLO violations.
Step 4: Deployment Gating

Error budgets will gate deployments based on performance:

Monitor Before Deployments: Ensure that error rates are within acceptable limits before initiating a deployment.
Automate Rollbacks: If an SLO breach occurs, automatically roll back changes to the previous stable state and notify the responsible team.
Step 5: Reporting and Feedback Loops

Regularly report on API performance and share insights with teams:

Generate Reports: Create dashboards that display SLI metrics over time.
Feedback Mechanisms: Implement mechanisms for cross-functional teams to provide feedback and improve SLOs collaboratively.

#### Closing Summary
and Common Mistakes/Follow-Ups

This framework ensures that APIs are reliable, performant, and aligned with business objectives. Key takeaways include:

Standardizing SLIs per endpoint.
Using error budgets effectively to manage deployment risk.
Implementing distributed tracing for accurate SLO breach attribution.
Automating deployment gating based on performance.

#### Common mistakes
often involve not setting clear SLIs or failing to adequately monitor and adjust error budgets over time. Follow-up questions might explore specific tools used, integration strategies with existing infrastructure, or handling more complex scenarios involving microservices and inter-team dependencies.

**Comparison:**

```
Endpoint	SLI Metrics	Error Budget
/v1/users	Availability: 99.9%, Latency: 200ms, Correctness: 95%	3%
/v1/orders	Availability: 99.8%, Latency: 300ms, Correctness: 94%	2.5%
```

**Code:**

```python
# Example of a simple SLO breach detection function
def check_slo_breach(current_error_rate, budget):
    if current_error_rate > budget:
        return True
    return False

# Example error budget management logic
class ErrorBudgetManager:
    def __init__(self, initial_budget=5%):
        self.error_budget = initial_budget

    def update_budget(self, new_data):
        # Logic to adjust the budget based on new data
        pass

    def check_for_breach(self, current_error_rate):
        if self.check_slo_breach(current_error_rate, self.error_budget):
            print("SLO breach detected!")
            return True
        return False
```

**Example input / output:**

```text
Input:
- Current error rate for "/v1/users" endpoint: 2.8%
- Initial budget: 3%

Output:
- No SLO breach detected.

Process:
1. Check if current error rate exceeds the initial budget.
2. Since 2.8% < 3%, no breach is detected.
```

**Data:**

```json
{
  "service_name": "API-Example",
  "endpoints": [
    {
      "endpoint_url": "/v1/users",
      "sli_metrics": {
        "availability": 99.9%,
        "latency": 200ms,
        "correctness": 95%
      },
      "error_budget": 3%
    },
    {
      "endpoint_url": "/v1/orders",
      "sli_metrics": {
        "availability": 99.8%,
        "latency": 300ms,
        "correctness": 94%
      },
      "error_budget": 2.5%
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. API Request
4. Tracing Span Creation
5. Error Rate > Error Budget?
6. SLO Breach Detected!
7. Monitor and Adjust Budgets
8. Rollback Changes, Notify Team

