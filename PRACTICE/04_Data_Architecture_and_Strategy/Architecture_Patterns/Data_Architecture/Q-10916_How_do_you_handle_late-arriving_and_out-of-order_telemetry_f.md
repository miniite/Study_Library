---
type: "practice"
difficulty: "Expert"
---

# How do you handle late-arriving and out-of-order telemetry for SLO/error-budget reporting where the numbers feed contractual SLAs and must be legally defensible? What watermarking, restatement, and reconciliation architecture do you build?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Reproducible Telemetry State](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Reproducible_Telemetry_State.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

Handling late-arriving and out-of-order telemetry data is critical when reporting Service Level Objectives (SLOs) and error budgets, especially in environments where these metrics must meet legal standards and contractual obligations. The solution involves a combination of watermarking, reprocessing, and reconciliation strategies to ensure the accuracy and integrity of reported data.

#### Triaging the Question

The core challenge is ensuring that telemetry data is processed correctly even when it arrives late or out of order. The question probes your understanding of robust data pipelines, watermarking techniques, and how to handle backfilling and reprocessing in a way that maintains legal defensibility. We will assume you have access to historical data, but new data may arrive asynchronously.

#### First Principles Solution
1. Watermarking

Watermarking is the practice of marking each piece of telemetry with its collection timestamp. This metadata helps track when and from where data was collected, ensuring that late-arriving data can be processed correctly.

PLAINTEXT
Input: [timestamp, value] -> [watermark, timestamp, value]
2. Watermark-Based Processing Pipeline

The pipeline should be designed to consume telemetry data based on the watermark rather than a strict chronological order. This ensures that late-arriving data does not disrupt earlier processed data.

PLAINTEXT
Data Flow:
Telemetry -> Watermark -> Watermarking Module -> Processing Queue -> SLO Calculation Engine
3. Reconciliation and Backfilling

Late-arriving telemetry needs to be reprocessed and reconciled with the existing data. A backfill mechanism can help ensure that all data points are accounted for, even if they come in out of order.

PLAINTEXT
Late Data -> Watermark -> Backfill Module -> Aggregation Engine -> SLO Calculation Engine
4. Legal Defensibility

To ensure legal defensibility, metadata and processing logs should be retained for audit purposes. This includes timestamps, watermark values, reprocessing records, and any anomalies detected during data flow.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key to handling late-arriving and out-of-order telemetry is a robust data architecture that accounts for asynchronous data flows. By implementing watermarking, ensuring proper processing order, and maintaining comprehensive logs, you can build a system that is both accurate and legally defensible.

#### Common mistakes
include:

Ignoring Watermarking: Relying solely on chronological order can lead to incorrect SLO calculations.
Lack of Backfill Mechanism: Failing to reprocess late-arriving data accurately.
Insufficient Logging: Not retaining enough metadata for audit purposes.

For further discussion, consider how you might integrate machine learning models to predict and correct anomalies in real-time or explore more advanced watermarking techniques that can handle even more complex data flow scenarios.

**Comparison:**

| Timestamp          | Value | Watermark                  |
|--------------------|-------|----------------------------|
| 2023-10-05T10:00   | 98    | 2023-10-05T10:00           |
| 2023-10-06T14:00   | 99    | 2023-10-06T14:00           |
| 2023-10-07T08:00   | 100   | 2023-10-07T08:00           |

**Code:**

```python
# Example of a simple watermarking function
def watermark_data(telemetry):
    return [(data[0], data[1], datetime.now()) for data in telemetry]
```

**Example input / output:**

```text
Input:
Telemetry: [("2023-10-05T10:00:00Z", 98), ("2023-10-06T14:00:00Z", 99), ("2023-10-07T08:00:00Z", 100)]

Watermarking:
[(datetime(2023, 10, 5, 10, 0, 0), 98),
 (datetime(2023, 10, 6, 14, 0, 0), 99),
 (datetime(2023, 10, 7, 8, 0, 0), 100)]

Processed:
SLO Calculation: [98, 99, 100]
```

**Data:**

```json
{
  "telemetry": [
    ["2023-10-05T10:00:00Z", 98],
    ["2023-10-06T14:00:00Z", 99],
    ["2023-10-07T08:00:00Z", 100]
  ]
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. Telemetry[Telemetry Data] --> Watermarking(Watermark each data point);
4. Watermarking --> Processing Queue[Processing Queue (Order by Watermark)];
5. Processing Queue --> SLO Calculation[SLO Calculation Engine];
6. LateData[Late/Out-of-Order Data] --> BackfillModule[Backfill Module (Reprocess)] --> AggregationEngine[Aggregation Engine];
7. AggregationEngine --> SLO Calculation;

