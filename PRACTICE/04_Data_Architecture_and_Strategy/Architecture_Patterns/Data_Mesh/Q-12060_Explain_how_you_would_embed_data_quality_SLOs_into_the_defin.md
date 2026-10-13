---
type: "practice"
difficulty: "Expert"
---

# Explain how you would embed data quality SLOs into the definition of a data product and hold domain teams accountable to them at scale.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Product Discoverability](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Mesh/Data_Product_Discoverability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
Embedding data quality Service Level Objectives (SLOs) directly within the definition of a data product in a Data Mesh architecture ensures that domain teams are held accountable for maintaining high-quality data at scale. This involves defining clear, measurable SLOs, integrating monitoring and alerting mechanisms, and leveraging automated tools to enforce compliance.

#### Triage
The question seeks an understanding of how to embed data quality standards into the governance framework of a Data Mesh setup, ensuring that domain teams can manage their data products effectively while maintaining consistency across the organization. Key assumptions include having a mature Data Mesh architecture with well-defined data domains and access control mechanisms.

Define Clear SLOs: Start by defining specific, measurable SLOs for each data product. These should cover key dimensions like accuracy, completeness, freshness, and consistency.

Integrate Monitoring Tools: Use tools that can continuously monitor adherence to these SLOs. This includes setting up automated tests, data quality checks, and anomaly detection mechanisms.

Automate Enforcement Mechanisms: Implement policies and automation routines that enforce compliance with the defined SLOs. This might involve triggering alerts for deviations from agreed-upon standards or automatically correcting issues if possible.

Domain Team Accountability: Ensure that domain teams have visibility into data quality metrics and are held accountable through regular reporting, dashboards, and performance evaluations linked to these metrics.

Feedback Loops: Establish robust feedback loops where insights from monitoring can drive continuous improvement in data quality practices within each domain.

#### Closing Summary
By embedding SLOs directly into the definition of a data product and leveraging Data Mesh principles, we create a scalable framework that ensures high data quality while maintaining operational efficiency across diverse teams. Common mistakes include not defining clear metrics or failing to integrate monitoring effectively, which can lead to suboptimal outcomes.

#### Common Mistakes

Lack of Clarity in Metrics: Poorly defined SLOs can lead to ambiguity and inconsistency.
Insufficient Monitoring: Without continuous and automated monitoring, issues may go unnoticed until they become critical.
Over-reliance on Alerts: Failing to integrate automated correction mechanisms can result in manual intervention bottlenecks.

**Comparison:**

```
Metric	Threshold (SLO)	Current Value	Alert
Accuracy	95%	93%	Yes
Completeness	90%	85%	Yes
Freshness (hourly)	<1 hour	1.5 hours	Yes
Consistency	98%	96%	No
```

**Code:**

```python
# Example of defining SLOs using a hypothetical framework
def define_slos(data_product):
    slos = {
        "accuracy": 95,
        "completeness": 90,
        "freshness": {"hourly": 1, "daily": 24},
        "consistency": 98
    }
    return slos

# Example of integrating monitoring and alerting using a hypothetical library
def monitor_slo(slos):
    # Simulate data quality checks
    current_metrics = {
        "accuracy": 93,
        "completeness": 85,
        "freshness": {"hourly": 1.5, "daily": 24},
        "consistency": 96
    }

    for metric, threshold in slos.items():
        if isinstance(threshold, int):
            if current_metrics[metric] < threshold:
                print(f"Alert: {metric} is below SLO")
        else:
            for key, value in threshold.items():
                if current_metrics["freshness"][key] > value:
                    print(f"Alert: Freshness is above threshold at {key}")
```

**Example input / output:**

```text
Input:
{
  "data_product": {
    "name": "Sales Data",
    "slos": {
      "accuracy": 95,
      "completeness": 90,
      "freshness": {"hourly": 1, "daily": 24},
      "consistency": 98
    }
  },
  "metrics": {
    "accuracy": 93,
    "completeness": 85,
    "freshness": {"hourly": 1.5, "daily": 24},
    "consistency": 96
  }
}

Output:
"Alert: Accuracy is below SLO"
"Alert: Completeness is below SLO"
"Freshness is above threshold at hourly"
```

**Data:**

```json
{
  "data_product": {
    "name": "Sales Data",
    "slos": {
      "accuracy": 95,
      "completeness": 90,
      "freshness": {"hourly": 1, "daily": 24},
      "consistency": 98
    }
  },
  "metrics": {
    "accuracy": 93,
    "completeness": 85,
    "freshness": {"hourly": 1.5, "daily": 24},
    "consistency": 96
  }
}
```

**Flow:**

1. Yes
2. Define SLOs
3. Is Data Product Monitored?
4. Monitor and Alert
5. Automate EnforcementMechanisms
6. Report and Evaluate
7. Continuous Improvement

