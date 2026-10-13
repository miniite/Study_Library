---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd architect feature freshness SLAs that compose across a chain: raw ingestion to silver to feature to online store to model. Where does a single flaky upstream silently violate a model's freshness guarantee, and who's paged?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Product SLAs](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Data_Product_SLAs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect feature freshness SLAs that compose across a chain from raw ingestion to silver to feature storage to online store to model, we need a robust data architecture strategy that ensures end-to-end visibility and control over data quality and freshness. This involves designing systems with clear interfaces, well-defined responsibilities, and comprehensive monitoring mechanisms.

The key challenge is to identify where any single flaky upstream can silently violate the model's freshness guarantee. We will use a mesh of data lakes and warehouses, leveraging different layers for raw data ingestion, transformed silver tables, and feature storage. Each layer has specific SLAs that need to be enforced, and these must cascade down to ensure the model receives fresh and reliable features.

First, let’s break down each component:

Raw Ingestion Layer: This is where data from various sources (e.g., databases, APIs, IoT devices) are ingested into a raw storage layer.
Silver Tables: Here, we clean, transform, and enrich the raw data to prepare it for feature computation.
Feature Storage: Raw and transformed features are stored in a way that supports efficient querying and updating, such as in a time-series database or a key-value store.
Online Store: This layer serves the most recent versions of features directly to online services and models.
Model Layer: The model uses the features from the online store for predictions.

#### Triage
What We’re Probing
SLA Enforcement: How we ensure that each step meets its SLAs, especially in a chain.
Latency Control: Managing how long data sits between different layers without affecting freshness guarantees.
Error Handling and Alerts: Mechanisms to detect and alert on failures or breaches of SLAs.

#### Assumptions
We have a reliable system for ingesting raw data with consistent intervals.
Transformation steps are defined and tested.
Feature storage supports efficient updates and queries.
The model has defined latency requirements.

#### First-Principles Solution
1. Raw Ingestion Layer

Ingestion should be asynchronous to handle high throughput and avoid blocking the system. We can use tools like Apache Kafka or AWS Kinesis for real-time streaming. To ensure freshness, we define a SLA that raw data must arrive within a certain time frame (e.g., 5 minutes).

2. Silver Tables

Transformations should be idempotent to allow reprocessing without affecting results. Use a batch processing system like Apache Beam or AWS Glue to handle the transformations in a reliable manner. Ensure that transformed data is stored with timestamps and versioning, allowing us to track changes.

3. Feature Storage

Use time-series databases (e.g., InfluxDB) or key-value stores (e.g., Redis) for storing features. These systems allow efficient querying of recent values while maintaining a history. Define SLAs that transformed data must be available within a specific timeframe (e.g., 10 minutes).

4. Online Store

Online store should serve the most recent version of features to online services and models. Use a caching layer like Redis to reduce latency, but still track the freshness of cached features. Define an SLA that features in the online store must be fresh within certain bounds (e.g., 15 minutes).

5. Model Layer

The model should consume features from the online store with known freshness guarantees. If a feature is outdated, it should have a fallback mechanism or use older data with a warning.

Monitoring and Alerts

To ensure compliance with SLAs, we need to implement comprehensive monitoring:

Latency Metrics: Track the latency of each transformation and storage step.
SLA Violation Alerts: Set up alerts for any step that breaches its SLA. For example, if raw data does not arrive within 5 minutes, or transformed data is unavailable after 10 minutes.
Error Handling: Implement robust error handling to recover from failures and ensure data integrity.

#### Closing Summary

By designing a mesh of data lakes and warehouses with clear layers and well-defined SLAs, we can ensure that feature freshness SLAs are maintained across the entire chain. Key steps include robust ingestion, reliable transformations, efficient storage, timely availability in the online store, and comprehensive monitoring and alerting. Any flaky upstream layer can silently violate a model's freshness guarantee if not properly managed.

#### Common Mistakes/Follow-Ups
Ignoring Latency: Ensuring that all layers meet their SLAs is crucial; ignoring latency issues can lead to stale data.
Overlooked Error Handling: Failing to implement robust error handling and fallback mechanisms can result in model degradation.
Inadequate Monitoring: Poor monitoring can fail to detect breaches of SLAs, leading to unknown data freshness issues.

**Comparison:**

```
Layer	SLA Definition	Monitoring Metrics
Raw	<5 minutes latency	Latency of raw data arrival
Silver	<10 minutes availability	Availability of transformed data
Feature	Fresh within 15 minutes	Query response time, versioning
Online	Fresh within 30 seconds	Caching hit rate, fallbacks
Model	Always fresh	Alert on stale feature usage
```

**Code:**

```python
# Example Python code for setting up a simple alert system using Prometheus and Grafana

from prometheus_client import start_http_server, Gauge

def setup_alerts():
    # Setup Prometheus metrics
    start_http_server(8000)
    raw_data_latency = Gauge('raw_data_latency', 'Latency of raw data arrival')
    transformed_data_availability = Gauge('transformed_data_availability', 'Availability of transformed data')

    def check_slas():
        # Simulate checking SLAs and setting alerts
        if raw_data_latency.value() > 300:  # More than 5 minutes
            print("Raw data latency alert!")
        if transformed_data_availability.value() < 1200:  # Less than 10 minutes
            print("Transformed data availability alert!")

    return check_slas

check_slas = setup_alerts()
```

**Example input / output:**

```text
Input:
- Raw data arrives at the ingestion layer.

Output:
- Raw data is ingested and transformed within SLAs.
- Transformed data is stored in a time-series database.
- Feature freshness is maintained across all layers.
- Alerts are triggered if any step breaches its SLA.
```

**Data:**

```json
{
  "raw_data_latency": {
    "average": 350,
    "max": 420
  },
  "transformed_data_availability": {
    "average": 1190,
    "max": 1280
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Raw Ingestion
8. Latency > 5 min?
9. Alert Raw Data Latency
10. Silver Transformations
11. Availability < 10 min?
12. Alert Transformed DataAvailability
13. Feature Storage
14. Freshness > 15 min?
15. Online Store Serving FreshFeatures
16. Fallback to Older Data withAlert
17. Model Uses Fresh Features

