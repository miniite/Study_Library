---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd architect the closed-loop system to reason about business impact (revenue, SLA penalties) not just technical metrics when choosing an action.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Business Aware Automation](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Closed_Loop_Automation/Business_Aware_Automation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In a closed-loop automation architecture, it's crucial to consider both technical and business impacts when deciding on actions. To achieve this, we design a system that integrates real-time financial and SLA data with our monitoring and response mechanisms. The key is to build a robust model that can interpret these metrics and make informed decisions based on a weighted score or priority.

#### Triaging the Question

The core of the question revolves around designing an architecture that factors in business-critical information (such as revenue, SLA penalties) alongside technical metrics when making automation decisions. The interviewer is probing our ability to integrate multiple data sources, handle complex decision-making logic, and ensure the system operates efficiently.

#### Assumptions
We are dealing with a cloud-native environment.
Data can be collected in real-time from various sources (e.g., financial systems, monitoring tools).
The architecture must support dynamic scaling and resilience.
We have access to historical data for training predictive models.

#### First Principles Solution

Data Collection: Gather data from diverse sources including finance, SLA compliance, technical metrics from monitoring tools like Prometheus or CloudWatch. Ensure these are ingested into a central data store (e.g., AWS Kinesis Data Firehose, Apache Kafka).

Centralized Data Store: Use a time-series database (like InfluxDB) to store and query the data efficiently. This allows us to run complex queries over historical data.

Modeling Business Impact: Develop machine learning models that can score business impact based on various metrics. For example, we could use a weighted scoring system where financial losses due to downtime are given higher priority than minor performance issues.

Decision-Making Logic: Implement decision-making logic that uses the scores from our ML models. This could be a custom Python script or a microservice running in Kubernetes. The logic should be flexible enough to handle different business rules and priorities, such as prioritizing revenue-generating services over less critical ones during peak hours.

Feedback Loop: Integrate a feedback mechanism where actions taken by the system are monitored for their actual impact on the business metrics. Use this data to continuously improve our models and decision-making process.

Automated Actions: Based on the scores, trigger automated actions (e.g., rolling back deployments, initiating failover processes) through orchestration tools like Kubernetes or Ansible.

#### Closing Summary
+ Common Mistakes

In summary, a robust closed-loop automation system should seamlessly integrate business-critical data with technical metrics to make informed decisions. Key challenges include ensuring real-time data collection, developing accurate predictive models, and maintaining dynamic decision-making logic that can adapt to changing priorities. Common mistakes might include over-reliance on technical metrics alone, neglecting to build in a feedback loop for continuous improvement, or failing to handle edge cases where business impact might be ambiguous.

Self-Check

Correctness + completeness confirmed.

**Comparison:**

| Service | SLA Violation | Revenue Loss | Impact Score |
|---------|---------------|--------------|-------------|
| web     | 5             | 0.2          | 3.6         |
| db      | 8             | 0.15         | 4.9         |

**Code:**

```python
# Example of a simple ML model that scores SLA and revenue impact
from sklearn.linear_model import LinearRegression

def train_ml_model(data):
    # Assume data contains columns ['sla_violation', 'revenue_loss']
    X = data[['sla_violation', 'revenue_loss']]
    y = data['impact_score']
    model = LinearRegression()
    model.fit(X, y)
    return model
```

**Example input / output:**

```text
Input:
Timestamp: 2023-10-01T14:00:00Z
Metrics: [{'service': 'web', 'sla_violation': 5, 'revenue_loss': 0.2}, {'service': 'db', 'sla_violation': 8, 'revenue_loss': 0.15}]

Output:
Impact Score (web): 3.6
Impact Score (db): 4.9

Decision: Prioritize action on db due to higher impact score.
```

**Data:**

```json
{
  "data": {
    "timestamp": "2023-10-01T14:00:00Z",
    "metrics": [
      {"service": "web", "sla_violation": 5, "revenue_loss": 0.2},
      {"service": "db", "sla_violation": 8, "revenue_loss": 0.15}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Data Collection
6. Is Real-Time Data Available?
7. Ingest Data into Central Store
8. Wait for Next Batch Window
9. Process and Analyze Data
10. Model Trained?
11. Generate Impact Scores
12. Train Model, then GenerateScores
13. Make Automated Decisions
14. Execute Action
15. Monitor Outcome andFeedback

