---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd set the org's reliability tiering for ML services so a fraud model gets multi-region tested failover while an internal analytics model consciously accepts single-region, and how you get data-science teams to accept a lower tier.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ SLOs and Error Budgets](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Availability_Zone_Resilience/SLOs_and_Error_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set the organization's reliability tiering for ML services, we need to ensure that critical models like fraud detection are robustly tested across multiple regions, while less critical models such as internal analytics can operate with a single-region setup. This involves understanding the specific requirements of each model and aligning them with the appropriate SLAs and operational costs.

#### Triage

The question is asking how to implement a tiered reliability strategy for ML services based on their criticality. The interviewer wants to understand your ability to balance reliability, cost-efficiency, and data science team buy-in. Assumptions include that we have a clear understanding of the business needs and SLAs for each model.

Step-by-Step Solution

Identify Key Models:

Fraud Detection Model (critical): Multi-region testing with failover capabilities.
Internal Analytics Model (secondary): Single-region deployment to save costs while maintaining basic availability.

Define Service Level Objectives (SLOs):

For the fraud detection model, define high SLOs for accuracy and uptime, requiring multi-region redundancy.
For internal analytics, lower SLOs are acceptable due to its secondary role in the organization's critical operations.

Architecture Design:

Fraud Detection Model:
Deploy multiple instances of the ML service across different AWS regions (e.g., US-East-1 and US-West-2).
Implement a centralized control plane that can dynamically redirect traffic based on region health checks.
Use Kubernetes for managing deployments and rolling updates to ensure high availability.
Internal Analytics Model:
Deploy a single instance in a cost-effective region, such as an AWS region with lower costs or even on-premises if the data is not sensitive.

Operational Costs:

For fraud detection, consider the increased operational costs of multi-region deployments but justify them by highlighting the potential financial losses from fraud.
For internal analytics, prioritize cost savings while ensuring basic availability and performance.

Team Buy-In:

Communicate the rationale behind different tiers to the data science teams.
Use real-world examples where single-region failures led to downtime or data loss in other models.
Provide case studies of how multi-region deployments have helped prevent larger outages in similar models.
Offer training sessions on best practices for managing SLOs and reliability in AWS.

Continuous Monitoring & Improvement:

Implement monitoring tools (e.g., CloudWatch, Prometheus) to continuously track service health.
Use A/B testing techniques to validate the performance of different tiers before fully committing resources.

#### Closing Summary

By setting up a tiered reliability strategy based on the criticality and business impact of each ML model, we can ensure that fraud detection is robust while saving costs where appropriate. Engaging with data science teams through clear communication and tangible examples will help secure their buy-in for this approach.

#### Common Mistakes/Follow-Ups
Mistake: Overdeploying resources to all models without considering the actual business impact.

#### Follow-Up
Highlight how tiered reliability can be flexible but must align with business priorities.
Mistake: Failing to communicate the trade-offs effectively to data science teams.

#### Follow-Up
Discuss potential resistance and strategies for addressing it, such as offering incentives or additional support.

**Comparison:**

```
Tier	Model Type	Deployment Region(s)	Cost Consideration	SLA Accuracy	SLA Availability
1	Fraud Detection	US-East-1, US-West-2	High	99.9%	99.5%
2	Internal Analytics	US-East-2	Low	98%	95%
```

**Code:**

```python
# Example Kubernetes YAML for multi-region deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fraud-detection-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: fraud-detection
  template:
    metadata:
      labels:
        app: fraud-detection
    spec:
      containers:
      - name: fraud-model-container
        image: <fraud-detection-image>
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: fraud-detection-service
spec:
  selector:
    app: fraud-detection
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

**Example input / output:**

```text
Input:
- Fraud Detection Model deployed across US-East-1 and US-West-2 regions.
- Internal Analytics Model deployed in a single region.

Output:
- Fraud Detection Model meets high SLOs with multi-region failover capability.
- Internal Analytics Model operates at lower costs while maintaining basic availability.
```

**Data:**

```yaml
# Example SLOs for Fraud Detection Model
fraud-detection-slo:
  accuracy: 99.9%
  availability: 99.5%

# Example SLOs for Internal Analytics Model
internal-analytics-slo:
  accuracy: 98%
  availability: 95%
```

**Flow:**

1. Multi-region Fraud Detection
2. Failover Capabilities
3. SLO Monitoring &Management
4. Single-region InternalAnalytics
5. Cost Optimization

