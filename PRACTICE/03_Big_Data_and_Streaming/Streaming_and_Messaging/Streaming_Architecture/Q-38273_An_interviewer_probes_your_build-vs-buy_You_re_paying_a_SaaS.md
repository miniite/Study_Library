---
type: "practice"
difficulty: "Expert"
---

# An interviewer probes your build-vs-buy: 'You're paying a SaaS observability vendor millions and also running a streaming platform. Where's the line between what you build and what you buy, and how does it shift as you scale?' Defend the boundary.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Streaming for AIOps](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Streaming_for_AIOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an AIOps architecture, deciding between building vs buying involves a careful balance of cost, control, flexibility, and risk management. When paying a SaaS observability vendor millions, you're investing in a managed service that provides critical monitoring capabilities while offloading maintenance and support. The line between what to build and what to buy is dynamic and depends on the specific needs, scale, and strategic goals of your organization.

The key idea here is to leverage external tools for core functionalities where they offer significant value in terms of scalability, reliability, and expertise, but still maintain control over critical parts of the stack that impact your unique business logic. This approach allows you to focus resources on differentiating aspects rather than reinventing the wheel.

#### Triaging the Question

The interviewer is probing the principles behind choosing between building from scratch versus leveraging existing solutions in a real-time streaming architecture. They are interested in understanding how the decision-making process evolves with increasing scale and what trade-offs are involved.

#### Assumptions

The organization pays a SaaS observability vendor for their core monitoring needs.
There is an internal streaming platform being run, likely to handle specific processing or aggregation tasks that the SaaS does not cover.
The goal is to justify the boundary between self-built and purchased solutions in terms of cost, functionality, and risk.

#### First Principles Solution

Core Functionality vs. Edge Cases:

Core functionalities like real-time log aggregation, metric collection, anomaly detection, and alerting are often best handled by SaaS offerings due to their scalability, reliability, and expertise.
Edge cases or specialized processing, such as complex data transformations, custom alerting rules, or integration with unique internal systems, may require custom build efforts.

Cost and Control:

SaaS solutions offer predictable costs and minimal initial setup time but come with vendor lock-in risks and potential service disruptions.
Self-built systems provide more control over the technology stack, allowing for optimizations that align closely with business needs but require significant upfront investment and maintenance.

Scalability and Reliability:

For mission-critical applications, SaaS solutions often have inherent scalability benefits due to their infrastructure investments.
Self-built systems can be customized for specific workload patterns but may not offer the same level of reliability or support unless robust in-house capabilities are already established.

Risk Management:

SaaS solutions simplify risk management by shifting responsibility to the vendor, reducing potential downtime and data loss risks.
Custom build efforts introduce new risks such as maintaining security patches, dealing with unexpected failures, and ensuring compliance with internal policies.

Integration and Flexibility:

External tools can be easily integrated into existing workflows, leveraging their APIs and SDKs for rapid deployment.
Self-built systems require more effort in integration but offer greater flexibility to tailor solutions directly to business needs.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the line between building and buying is often a trade-off between cost, control, scalability, reliability, and risk. When paying a SaaS observability vendor millions, it’s prudent to leverage their offerings for core functionalities that they excel at while retaining internal capabilities where custom solutions are essential.

#### Common mistakes
include over-customizing SaaS tools or underestimating the maintenance burden of self-built systems. Follow-ups might explore specific scenarios, such as data security requirements or integration complexities between different services.

**Comparison:**

```
Metric	Value	Timestamp
cpu_usage	75	2023-10-01T14:00:00Z
memory_usage	60	2023-10-01T14:00:00Z
```

**Code:**

```python
# Example of integrating a SaaS observability API
import requests

def fetch_metrics_from_saastracker(api_key):
    url = "https://api.saastracker.com/v1/metrics"
    headers = {"Authorization": f"Bearer {api_key}"}

    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        return response.json()
    else:
        print(f"Error fetching metrics: {response.status_code}")
```

**Example input / output:**

```

Input:

PYTHON
Copy
api_key = "YOUR_API_KEY_HERE"
metrics = fetch_metrics_from_saastracker(api_key)
print(metrics)

Output:

JSON
Copy
{
  "metrics": [
    {
      "metric_name": "cpu_usage",
      "value": 75,
      "timestamp": "2023-10-01T14:00:00Z"
    },
    {
      "metric_name": "memory_usage",
      "value": 60,
      "timestamp": "2023-10-01T14:00:00Z"
    }
  ]
}
```

**Data:**

```json
{
  "metrics": [
    {
      "metric_name": "cpu_usage",
      "value": 75,
      "timestamp": "2023-10-01T14:00:00Z"
    },
    {
      "metric_name": "memory_usage",
      "value": 60,
      "timestamp": "2023-10-01T14:00:00Z"
    }
  ]
}
```

**Flow:**

1. Start
2. Identify Core vs. Edge Cases
3. Choose SaaS for CoreFunctionality
4. Build Custom Solutions forEdge Cases
5. Integrate with ExistingWorkflows
6. Optimize and Tailor toBusiness Needs
7. Monitor Performance andReliability
8. Review Costs, Control,Scalability, Reliability, Risk
9. End

