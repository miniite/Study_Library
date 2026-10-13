---
type: "practice"
difficulty: "Expert"
---

# What guardrails do you put in place so 'you build it, you run it' for data products does not overwhelm domain teams with pager duty?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Domain Ownership vs Cross-cutting Data](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Mesh/Domain_Ownership_vs_Cross-cutting_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that "you build it, you run it" (YBIYROI) does not overwhelm domain teams with pager duties in a Data Mesh architecture, we need to establish robust guardrails and mechanisms that balance responsibility between development and operations while maintaining data quality and governance. The key is to delineate clear roles and responsibilities, ensuring that each team has the necessary tools and support to manage their data products effectively.

#### Triage
The question seeks to understand how to implement effective operational support systems for domain teams in a Data Mesh environment. It probes understanding of DevOps principles, responsibility boundaries, and the importance of governance in distributed data environments. Assumptions include that each domain team has a defined scope of responsibility over specific datasets or applications.

#### First-Principles Solution

Define Clear Boundaries: Establish clear ownership and responsibilities for data products across teams.
Automate Monitoring and Alerting: Implement automated monitoring and alerting to detect issues early, reducing the need for manual intervention.
Provide Self-Service Tools: Offer self-service tools and dashboards that empower domain teams to monitor their own data products.
Centralized Support Frameworks: Develop centralized support frameworks to handle common issues and provide on-demand assistance when needed.

Step-by-Step Solution:

Define Ownership: Clearly define which team owns each dataset or application, ensuring accountability for its lifecycle.
Automate Monitoring: Use tools like Prometheus, Grafana, or custom alerts to monitor critical metrics related to data quality, latency, and availability.
Self-Service Dashboards: Provide domain teams with access to dashboards that display key performance indicators (KPIs) relevant to their data products.
Centralized Support Mechanisms: Set up a centralized support team or helpdesk to handle common issues, triage critical alerts, and provide on-demand assistance.

#### Closing Summary
By implementing these guardrails, we can ensure that domain teams have the necessary tools and support to manage their data products effectively without being overwhelmed by pager duties. This approach balances accountability with operational efficiency, fostering a productive and scalable Data Mesh environment.

#### Common Mistakes/Follow-Ups

Overreliance on centralized operations: Domain teams should not be completely reliant on a single support team.
Lack of self-service tools: Empowering domain teams with self-monitoring capabilities can significantly reduce the load on support resources.
Insufficient training and documentation: Ensuring that all teams are well-informed about their responsibilities and available resources is crucial.

**Comparison:**

```
Metric	Description	Unit
Data Latency	Time taken to fetch data from the source	seconds
Data Quality Issues	Number of detected issues in dataset	count
Role	Responsibilities	Tools
Domain Team	Owns and maintains specific datasets	Self-service dashboards, monitoring tools
Support Team	Monitors system health, triages alerts	Prometheus, Grafana, Jira
```

**Code:**

```python
# Example code for automated alerting setup using Prometheus and Grafana
import prometheus_client

# Define metrics
metrics = {
    'data_latency': prometheus_client.Gauge('data_latency_seconds', 'Latency in fetching data'),
    'data_quality_issues': prometheus_client.Counter('data_quality_issues_total', 'Number of data quality issues detected')
}

def update_metrics(latency, issues):
    metrics['data_latency'].set(latency)
    metrics['data_quality_issues'].inc(issues)

# Integrate with Grafana for visualization
grafana_api_key = "YOUR_GRAFANA_API_KEY"
grafana_url = "http://grafana.example.com/api/datasources/proxy/1/query"

def push_to_grafana(query):
    payload = {
        "query": query,
        "time": int(time.time() * 1000)
    }
    response = requests.post(grafana_url, headers={"Authorization": f"Bearer {grafana_api_key}"}, json=payload)
    return response.json()
```

**Example input / output:**

```text
Input:
- Metrics data for monitoring data latency and data quality issues.
- Alert events triggered based on thresholds.

Output:
- Dashboard with real-time visualizations of data latency and data quality issues.
- Alerts pushed to centralized support system or directly to responsible domain teams.
```

**Data:**

```json
{
    "metrics": {
        "data_latency": 2.5,
        "data_quality_issues": 3
    },
    "events": [
        {"timestamp": "2023-10-01T14:00:00Z", "severity": "warning", "message": "High latency detected for dataset A"},
        {"timestamp": "2023-10-01T15:00:00Z", "severity": "error", "message": "Data quality issue found in dataset B"}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Data Product Owner
4. Is Data Product Critical?
5. Set Up Automated Monitoring
6. Define Alert Thresholds
7. Provide Self-Service Tools
8. Centralized Support Team
9. Handle Common Issues
10. Domain Team

