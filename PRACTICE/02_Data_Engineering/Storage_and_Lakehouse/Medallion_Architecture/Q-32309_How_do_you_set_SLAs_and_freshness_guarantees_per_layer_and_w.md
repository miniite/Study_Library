---
type: "practice"
difficulty: "Deep"
---

# How do you set SLAs and freshness guarantees per layer, and what happens to Gold's SLA when a Bronze source is delayed by six hours?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Medallion Layer Quality](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Medallion_Layer_Quality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a data pipeline using Medallion Architecture (Bronze/Silver/Gold), setting Service Level Agreements (SLAs) and freshness guarantees is crucial to ensure that users can rely on the data at different layers. This process involves understanding the specific SLA requirements for each layer, managing dependencies, and ensuring that changes in one layer do not negatively impact others.

#### Triage
What the Question is Really Asking

The question asks about setting SLAs and freshness guarantees across the Bronze, Silver, and Gold layers of a Medallion Architecture. Specifically, it focuses on how delays in the Bronze layer might affect the Golden layer's service level. The interviewer is probing your understanding of dependency management, data quality controls, and the implications of delayed sources.

#### Assumptions
Bronze Layer: Raw, unprocessed, and often sourced directly from external systems.
Silver Layer: Processed and cleaned but not yet analyzed. It serves as a staging area for further transformations.
Gold Layer: The final layer where data is fully transformed, curated, and ready for business intelligence (BI) or analytical use.

#### First Principles Solution

Define SLAs and Freshness Guarantees:

Bronze Layer: Raw data may have inherent delays due to external sources' schedules. For example, if the source updates every six hours, the bronze layer should reflect this.
Silver Layer: Data is cleaned and transformed but not yet optimized for analysis. The goal here is to minimize delays as much as possible while ensuring quality.
Gold Layer: Optimized for analytics, with the strictest SLAs. Data must be refreshed regularly to support real-time or near-real-time business insights.

Setting SLAs:

Bronze: Delay tolerance of six hours (aligning with the source's schedule).
Silver: Aim for a delay under an hour.
Gold: Ensure freshness every 15 minutes.

Handling Delays in Bronze Layer:

If the Bronze layer is delayed by six hours, we need to understand how this affects the Gold layer.
In most cases, delays in the Bronze layer should be absorbed or mitigated through robust data validation and reconciliation processes in Silver before reaching Gold.
However, if a significant delay occurs, it could impact the freshness of the Gold layer.

Impact on Gold Layer:

The primary concern is that any delay in the Bronze layer may cause the Silver layer to be stale or inaccurate.
If the Silver layer remains untainted (i.e., data cleaning processes are robust), then the Gold layer should still reflect a delayed but valid dataset.
In extreme cases, where the Silver layer also becomes delayed, the Gold layer’s freshness guarantees will naturally suffer.

Mitigation Strategies:

Implement buffer zones in the ETL pipeline to handle occasional delays.
Use event-time processing techniques to ensure that events are processed when they occur, not necessarily at fixed intervals.
Perform data validation and reconciliation between layers to ensure consistency even if there are delays upstream.

Monitoring and Alerts:

Set up real-time monitoring for each layer to detect delays early.
Trigger alerts when SLAs are breached to allow proactive remediation.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Setting appropriate SLAs and freshness guarantees is a critical aspect of Medallion Architecture. The key challenge lies in managing dependencies between layers while ensuring robust data quality controls. Common mistakes include:

Overlooking delays in Bronze layer, leading to cascading issues.
Failing to implement effective monitoring and alerts.
Not having buffer zones or event-time processing mechanisms.

In conclusion, the solution involves defining clear SLAs for each layer, handling dependencies proactively, and ensuring robust data validation processes. The impact of delayed Bronze layers on Gold layers should be managed through careful design and proactive monitoring.

#### Follow-ups
might include:

Discussing specific tools and technologies used in these practices.
Exploring strategies for real-time vs. batch processing.
Detailing the trade-offs between different SLA settings based on business needs.

**Comparison:**

| Layer | SLA Definition                | Freshness Guarantees |
|-------|-------------------------------|----------------------|
| Bronze| Delay tolerance of 6 hours    | N/A                  |
| Silver| Under an hour                 | High                 |
| Gold  | Every 15 minutes              | Real-time            |

**Code:**

```python
# Example of setting up an alert system in Python using pandas and a notification service
import pandas as pd
from datetime import datetime, timedelta

def check_sla(dataframe):
    current_time = datetime.now()
    delayed_threshold = current_time - timedelta(hours=6)

    # Check for delays in Bronze layer
    bronze_delays = dataframe['bronze_timestamp'] < delayed_threshold

    if bronze_delays.any():
        print("Warning: Delay detected in Bronze layer. Initiating mitigation steps.")
        # Implement mitigation logic here

# Example DataFrame representing SLA checks
data = {'bronze_timestamp': pd.date_range(start='2023-10-05', periods=6, freq='H')}
df = pd.DataFrame(data)

check_sla(df)
```

**Example input / output:**

```text
Input:
Bronze Layer Data:
Timestamp: 2023-10-05T00:00:00, 2023-10-05T06:00:00, 2023-10-05T12:00:00

Output:
Silver Layer Data (after cleaning and processing):
Timestamp: 2023-10-05T01:00:00, 2023-10-05T07:00:00, 2023-10-05T13:00:00

Gold Layer Data (after final transformations):
Timestamp: 2023-10-05T02:00:00, 2023-10-05T08:00:00, 2023-10-05T14:00:00
```

**Data:**

```json
{
  "bronze_layer": {
    "source": "External System",
    "delay_tolerance": "6 hours"
  },
  "silver_layer": {
    "processing_timeframe": "Under an hour",
    "quality_control": true
  },
  "gold_layer": {
    "freshness_period": "15 minutes"
  }
}
```

**Flow:**

1. Raw Data
2. Bronze Layer (Delayed by 6hours)
3. Silver Layer
4. Gold Layer (FreshnessGuaranteed)

