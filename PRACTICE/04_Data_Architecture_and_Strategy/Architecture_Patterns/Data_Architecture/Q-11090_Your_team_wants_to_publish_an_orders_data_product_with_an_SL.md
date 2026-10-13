---
type: "practice"
difficulty: "Intermediate"
---

# Your team wants to publish an orders data product with an SLA. What would you actually promise consumers about freshness, availability, and correctness, and how do you measure whether you're meeting it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Product SLAs](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Data_Product_SLAs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to define what "freshness," "availability," and "correctness" mean in the context of a data product like orders data. We'll then outline how these promises can be translated into specific SLA targets and the mechanisms for measuring compliance.

#### Triaging the Question

The interviewer is probing your understanding of key data quality attributes and how to operationalize them as service level agreements (SLAs). They are also interested in how you would architect a system to meet these SLAs, considering factors like data pipelines, storage strategies, and monitoring tools. Key assumptions include:

The team has access to reliable input data.
There is no external dependency that could affect the delivery of this data product.
Defining Key Attributes
Freshness: This refers to how recent the data is in relation to when it was produced or updated.
Availability: This pertains to whether the data can be accessed at any given time without failures.
Correctness: This ensures that the published data matches the actual state of the business.
Setting SLAs

For an orders data product, let's set some practical SLA targets:

Freshness:
We promise a latency of no more than 15 minutes between order completion and its appearance in the data warehouse.
Availability:
The service must be available at least 99.9% of the time (an uptime guarantee).
Correctness:
Orders should only ever be updated once, with a maximum error rate of 0.1%.
Measurement Mechanisms

To measure whether we are meeting these SLAs, we need to implement several monitoring and validation processes.

Freshness
Implement a data freshness dashboard that shows the current latency for each order entry.
Set up alerts when this exceeds our promised threshold.
Availability
Use a service level objective (SLO) tool like Prometheus or Datadog to monitor uptime metrics.
Define SLOs at the API and microservice levels, ensuring high availability throughout the stack.
Correctness
Develop a data validation pipeline that checks for inconsistencies against historical data.
Implement checksums or hash functions on critical fields to detect potential issues.

#### Example
SLA Metrics

To operationalize these SLAs:

Freshness:

Track average latency using a time-series database like TimescaleDB.
Define alerts in monitoring tools when the average latency exceeds 15 minutes.

Availability:

Monitor API uptime and microservice health using an SLO tool.
Set up failover mechanisms for critical services to ensure no downtime.

Correctness:

Use a data validation framework like Apache Beam or Flink to run periodic checks on the orders data.
Integrate these checks with alert systems that notify stakeholders when issues are detected.

#### Closing Summary
and Common Mistakes

In summary, setting SLAs for an orders data product involves defining clear targets for freshness, availability, and correctness. Ensuring compliance requires robust monitoring and validation mechanisms. Common mistakes include underestimating the complexity of these systems or ignoring edge cases such as network outages or data corruption.

When implementing these solutions, always consider the overall architecture to ensure that all components work harmoniously towards meeting the SLAs.

**Comparison:**

| Metric | Target | Measurement |
|--------|--------|-------------|
| Freshness Latency | < 15 minutes | Time-series database, Alerts in monitoring tools |
| Availability Uptime | > 99.9% | SLO tool (Prometheus), API and microservice health checks |
| Correctness Error Rate | ≤ 0.1% | Data validation pipeline, Checksums on critical fields |

**Code:**

```python
# Example Python code snippet for setting up a freshness dashboard alert
import requests

def check_freshness():
    url = "http://orders-api.com/freshness"
    response = requests.get(url)
    if response.status_code != 200:
        print("Freshness Alert: API is down")
    else:
        latency = response.json()['latency']
        if latency > 15:
            print(f"Freshness Alert: Latency {latency} exceeds threshold")

check_freshness()
```

**Example input / output:**

```text
Input:
- Orders data with timestamps

Output:
- Freshness dashboard displaying latency < 15 minutes for orders
- Alerts triggered when latency > 15 minutes or API is down
- Data validation pipeline confirming order integrity
- Monitoring tools logging uptime and service health
```

**Data:**

```json
{
  "orders": [
    {
      "order_id": 1234,
      "customer_id": 5678,
      "status": "completed",
      "timestamp": "2023-10-01T14:30:00Z"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Data Source
4. Is Order Completed?
5. Write to Orders Warehouse
6. Queue for Backfill
7. Freshness Dashboard Update
8. Backfill Complete
9. Freshness Dashboard Update

