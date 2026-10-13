---
type: "practice"
difficulty: "Deep"
---

# What's your view on governing 'data as a product' contracts between a platform team and GenAI feature teams, and what SLAs belong in those contracts?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ GenAI and Warehouse Governance](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/GenAI_and_Warehouse_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In modern AI platforms, data acts not just as an input but as a product itself. The governance of "data as a product" (DaP) between platform teams and GenAI feature teams is crucial to ensure quality, security, and compliance. This governance must be formalized through Service Level Agreements (SLAs), which define the responsibilities, expectations, and performance metrics for data provision, usage, and protection.

At its core, SLAs are about setting clear boundaries and expectations that both parties must adhere to, ensuring seamless integration of data into GenAI features while maintaining high standards. Here’s a structured approach to defining these SLAs:

Data Quality: Ensure the data provided meets specific quality standards, such as accuracy, completeness, and consistency.
Latency and Availability: Define acceptable response times for data retrieval and the availability of data sources.
Security and Compliance: Establish protocols for handling sensitive or regulated data, including encryption, access controls, and audit trails.
Versioning and Backward Compatibility: Ensure that changes in data schema are managed smoothly to avoid breaking GenAI models.
Performance Metrics: Use metrics like throughput, error rates, and response times to measure performance.

#### Triage

The question is asking for a strategic view on SLAs governing the contract between platform teams providing data as a product and feature teams building GenAI applications. The key aspects are:

Understanding the roles and responsibilities of both parties.
Identifying critical performance metrics and quality standards.
Ensuring compliance with relevant regulations.

#### First-Principles Solution

Data Quality SLA:

Define minimum acceptable quality levels for data inputs, e.g., 95% accuracy, 99% completeness.
Include a process for reporting and fixing issues (e.g., within one business day).

Latency and Availability SLA:

Set response time thresholds, e.g., less than 1 second latency.
Ensure high availability with no more than 0.5% downtime.

Security and Compliance SLA:

Implement robust encryption (e.g., AES-256) for data at rest and in transit.
Enforce strict access controls, with role-based permissions.
Maintain audit trails for all data access and modifications.

Versioning and Backward Compatibility SLA:

Provide stable APIs with version control, ensuring backward compatibility.
Offer a clear deprecation policy and transition plan when changes are required.

Performance Metrics SLA:

Define key performance indicators (KPIs) such as response time, error rate, and throughput.
Set thresholds for these KPIs to ensure optimal performance.

#### Closing Summary

In summary, governing "data as a product" requires detailed SLAs that cover data quality, latency, security, versioning, and performance. These agreements ensure that both platform and feature teams operate effectively while maintaining high standards of data integrity and compliance.

#### Common Mistakes/Follow-Ups
Mistake 1: Ignoring data quality issues, leading to suboptimal AI model performance.
Mistake 2: Failing to address security concerns, compromising sensitive data.

#### Follow-Up
Discussing the impact of these SLAs on overall project timelines and costs.

**Comparison:**

```
Metric	Thresholds
Response Time	<1 second
Error Rate	<0.5%
Availability	>99.99%
```

**Code:**

```python
# Example SLA enforcement in a platform team's codebase
def enforce_data_quality(data):
    if not all([validate_accuracy(d) for d in data]):
        raise DataQualityException("Data accuracy below threshold")
```

**Example input / output:**

```text
Input:
- Data quality with accuracy at 93%, completeness at 98%
Output:
- SLA Violation: Accuracy threshold not met. Action required to improve data quality.
```

**Data:**

```json
{
  "data_quality": {
    "accuracy_threshold": 95,
    "completeness_threshold": 99
  },
  "latency_requirements": {
    "response_time_ms": <1000
  },
  "security_requirements": {
    "encryption_algorithm": "AES-256",
    "access_controls": ["role_based"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Data Quality Check
4. Pass?
5. Log Pass
6. Generate Alert and Fix Issues

