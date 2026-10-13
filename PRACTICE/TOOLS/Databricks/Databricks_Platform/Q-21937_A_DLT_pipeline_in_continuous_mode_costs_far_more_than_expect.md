---
type: "practice"
difficulty: "Deep"
---

# A DLT pipeline in continuous mode costs far more than expected. How do you decide between continuous and triggered mode for it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Streaming Ingest Tuning](<../../../../TOOLS/Databricks/Databricks_Platform/Streaming_Ingest_Tuning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a Data Lakehouse (DLT) pipeline costing more in continuous mode, we need to understand the trade-offs between continuous processing and triggered mode processing. Continuous mode provides real-time insights but can be resource-intensive, whereas triggered mode processes data on-demand with potentially higher flexibility and cost-effectiveness.

Firstly, let's clarify that DLT pipelines are typically implemented using Structured Streaming in Apache Spark within a Databricks environment. We need to analyze the costs associated with each mode and evaluate them based on our business requirements, such as data freshness, processing latency, and resource utilization.

**Comparison:**

```
Mode	Cost Factor	Example Scenario
Continuous	High CPU Utilization	Processing real-time market data
Triggered	Variable CPU Utilization	Batch processing ETL jobs with scheduled triggers

Given the cost analysis, continuous mode is more suitable for applications requiring high data freshness and low latency. For batch processing or less time-critical scenarios, triggered mode can provide significant cost savings while maintaining acceptable data latency.
```

**Code:**

```
MERMAID
Copy
graph TB
    A[Continuous Mode] --> B{Data Freshness Requirements}
    B --> C[High Freshness Required]
    B --> D[Low Freshness Tolerated]
    C --> E[Use Continuous Mode for Real-Time Insights]
    D --> F[Consider Triggered Mode with Scheduled Triggers]

    G[Triggered Mode] --> H{Scheduled Processing Frequency}
    H --> I[Very Frequent Scheduling]
    H --> J[Less Frequent Scheduling]
    I --> K[Use for Time-Critical Applications, e.g., Finance]
    J --> L[Use for Batch Processing, ETL Jobs]
```

**Example input / output:**

```text
**Example:**

| Mode      | Cost Factor            | Example Scenario                          |
|-----------|------------------------|------------------------------------------|
| Continuous | High CPU Utilization   | Processing real-time market data          |
| Triggered  | Variable CPU Utilization | Batch processing ETL jobs with scheduled triggers |

In the continuous mode, Databricks would maintain a Spark cluster running continuously to process incoming data streams. This setup ensures high data freshness but can be resource-intensive and costly.

In contrast, triggered mode allows for more flexibility by processing data on-demand based on pre-defined schedules. This approach is cost-effective but may introduce some latency in data insights.
```

**Data:**

```json
{
  "continuousCost": {
    "highCPUUtilization": true,
    "realTimeInsights": true,
    "highLatencyTolerance": false
  },
  "triggeredCost": {
    "variableCPUUtilization": true,
    "realTimeInsights": false,
    "lowLatencyTolerance": false
  }
}
```

**Flow:**

1. Continuous Mode
2. Data Freshness Requirements
3. High Freshness Required
4. Low Freshness Tolerated
5. Use Continuous Mode forReal-Time Insights
6. Consider Triggered Mode withScheduled Triggers
7. Triggered Mode
8. Scheduled ProcessingFrequency
9. Very Frequent Scheduling
10. Less Frequent Scheduling
11. Use for Time-CriticalApplications, e.g., Finance
12. Use for Batch Processing, ETLJobs

