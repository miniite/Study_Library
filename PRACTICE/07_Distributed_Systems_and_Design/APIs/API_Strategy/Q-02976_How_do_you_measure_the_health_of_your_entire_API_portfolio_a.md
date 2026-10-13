---
type: "practice"
difficulty: "Expert"
---

# How do you measure the health of your entire API portfolio as an executive scorecard? Define the 6-8 metrics (adoption, breakage rate, deprecation debt, DX/CSAT, security findings, cost-per-call) and how you'd act when one goes red.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Lifecycle Metrics and Retirement](<../../../../07_Distributed_Systems_and_Design/APIs/API_Strategy/API_Lifecycle_Metrics_and_Retirement.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To measure the health of an API portfolio as an executive scorecard, we need to consider a holistic view that includes both business outcomes and operational metrics. Six key metrics stand out: adoption, breakage rate, deprecation debt, developer experience (DX) and customer satisfaction (CSAT), security findings, and cost-per-call. Each metric provides critical insights into different aspects of API health.

Firstly, adoption measures the number of active users or applications consuming your APIs. High adoption indicates that your APIs are valuable to a broad user base. Secondly, the breakage rate tracks how frequently APIs fail due to bugs or issues. A high breakage rate suggests ongoing development and maintenance challenges. Thirdly, deprecation debt quantifies the number of deprecated APIs still in use. This metric highlights the need for timely updates and retirements. Fourthly, DX/CSAT (Developer Experience/Customer Satisfaction) measures user feedback on ease-of-use and performance. Poor DX can lead to high churn rates among developers and customers. Fifthly, security findings indicate potential vulnerabilities in your APIs. Regular security audits are essential for maintaining trust and compliance. Lastly, cost-per-call provides financial insight into the operational efficiency of your API infrastructure.

To act on these metrics when they go "red," we need a strategic approach. For instance:

If adoption is low, consider marketing or integration improvements to increase usage.
A high breakage rate might signal development process issues; improve testing and CI/CD practices.
Large deprecation debt indicates poor lifecycle management; prioritize retiring old APIs.
Poor DX/CSAT suggests usability problems; invest in better documentation and support.
Security findings should prompt immediate remediation actions to protect against breaches.
High cost-per-call might indicate inefficient infrastructure; explore cost optimization strategies.

**Comparison:**

| Metric          | Description                                                                                                                                                         | Ideal Range |
|-----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------|
| Adoption        | Percentage of active users consuming the API                                                                                                                          | High        |
| Breakage Rate   | Ratio of failed requests to total requests                                                                                                                            | Low         |
| Deprecation Debt| Number of deprecated APIs still in use                                                                                                                                | Zero or low |
| DX/CSAT         | User satisfaction and ease-of-use scores                                                                                                                              | High        |
| Security Findings| Number of security vulnerabilities identified                                                                     | Zero       |
| Cost-per-call   | Operational cost per API call, considering infrastructure costs                                                                                                     | Low         |

**Code:**

```python
def calculate_adoption(metrics):
    return metrics['active_users'] / total_possible_users

def calculate_breakage_rate(metrics):
    return metrics['failed_requests'] / total_requests

def calculate_deprecation_debt(metrics):
    return len(deprecated_apis_in_use)

def calculate_dx_csat(metrics):
    return average_satisfaction_score * 100

def calculate_security_findings(metrics):
    return len(security_vulnerabilities_found)

def calculate_cost_per_call(metrics):
    return total_api_calls * metrics['cost'] / sum_of_api_calls
```

**Example input / output:**

```text
Input:
- Active users: 5000 / Total possible users: 10000
- Failed requests: 200 / Total requests: 100000
- Deprecation debt: ["api/v1.0/user"]
- Average satisfaction score: 4.5/5
- Security vulnerabilities found: ["SQL Injection", "Cross-Site Scripting"]
- Cost per call: $0.002

Output:
Adoption: 50%
Breakage rate: 0.20%
Deprecation debt: 1 API
DX/CSAT: 90%
Security findings: 2 vulnerabilities found
Cost-per-call: $0.006 per call
```

**Data:**

```json
{
  "metrics": {
    "active_users": 5000,
    "total_possible_users": 10000,
    "failed_requests": 200,
    "total_requests": 100000,
    "deprecated_apis_in_use": ["api/v1.0/user"],
    "average_satisfaction_score": 4.5,
    "security_vulnerabilities_found": ["SQL Injection", "Cross-Site Scripting"],
    "cost": 0.002, // per call
    "sum_of_api_calls": 300000
  }
}
```

**Flow:**

1. Define Key Metrics
2. Calculate Adoption
3. Calculate Breakage Rate
4. Calculate Deprecation Debt
5. Calculate DX/CSAT
6. Calculate Security Findings
7. Calculate Cost-per-Call
8. Monitor Adoption
9. Monitor Breakage Rate
10. Manage Deprecation Debt
11. Enhance DX/CSAT
12. Address Security Risks
13. Optimize Costs

