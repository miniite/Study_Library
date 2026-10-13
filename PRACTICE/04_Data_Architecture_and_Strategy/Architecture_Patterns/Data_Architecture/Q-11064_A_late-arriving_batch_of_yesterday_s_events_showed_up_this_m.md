---
type: "practice"
difficulty: "Deep"
---

# A late-arriving batch of yesterday's events showed up this morning and your daily aggregate is now wrong. How do you design the pipeline so it handles late data instead of silently undercounting?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ End to End No Data Loss](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of handling late-arriving batches, we need to rethink our current architecture and ensure that it can accurately process both timely and delayed data without leading to undercounts in daily aggregates. The key is designing a robust system that respects the temporal nature of data and avoids losing any important information.

#### Triage
Understanding the Problem

The problem at hand involves late-arriving data, which means some events from yesterday have shown up today. This scenario can lead to incorrect daily aggregations if not handled properly. The interviewer is probing our understanding of modern data architecture practices, specifically how we handle temporal data and ensure consistency in aggregations.

#### Assumptions
Late Data Definition: Late data refers to records that arrive after the period they should have been processed.
Daily Aggregation Requirement: We need accurate daily aggregates despite late data.
Event Consistency: Each event has a timestamp indicating when it occurred, and we must respect this timestamp.
Step-by-Step Solution from First Principles
1. Data Ingestion Layer

The ingestion layer should be capable of distinguishing between timely and late data. This can be achieved by maintaining a watermark or timestamp for the latest batch processed.

Yes
No
Data Source
Ingestion Pipeline
Is data from latest batch?
(Insert into warehouse)
(Save to late-arrival buffer)
2. Late Arrival Buffer

A buffer is essential for storing late data temporarily until it can be processed in the context of its original time period.

Data Source
Late Arrival Buffer
Deduplication Logic
Late Batch Processor
3. Deduplication and Aggregation

To avoid double counting, deduplication logic is necessary to ensure each event is processed only once.

Late Arrival Buffer
Deduplication Logic
Aggregator
Warehouse
4. Aggregation Process

The aggregation process should be designed to include late data in the current day's batch, ensuring that no valuable information is lost.

Late Batch Processor
Deduplication Logic
Aggregator
Warehouse
5. Synchronization and Watermarking

A critical component of the architecture is maintaining a watermark that indicates the latest time period for which data has been processed.

Warehouse
Watermark Generator
Deduplication Logic
Aggregator

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing a multi-layered architecture that includes late arrival buffers, deduplication logic, and synchronized watermarks, we can ensure that our daily aggregates are accurate even in the presence of delayed data. Key mistakes include failing to buffer late data, ignoring temporal consistency, and not accounting for duplicate records.

Common follow-ups might involve discussing specific technologies like Apache Kafka for handling streaming data, or more advanced techniques such as incremental backups and point-in-time recovery.

**Comparison:**

| Event ID | Timestamp         | Value |
|----------|------------------|-------|
| 1        | 2023-10-04T23:59:59Z | 10    |
| 2        | 2023-10-05T00:01:00Z | 20    |

**Code:**

```python
# Example code snippet for watermark generation in a Data Warehouse

def update_watermark(current_time):
    global latest_watermark
    if current_time > latest_watermark:
        latest_watermark = current_time
        print(f"Watermark updated to {latest_watermark}")
```

**Example input / output:**

```text
Input:
- Current Watermark = '2023-10-05T14:30:00Z'
- New Data Timestamps = ['2023-10-04T23:59:59Z', '2023-10-05T00:01:00Z']

Output:
- Updated Watermark = '2023-10-05T00:01:00Z'
```

**Data:**

```json
{
  "latestWatermark": "2023-10-05T14:30:00Z",
  "bufferedData": [
    {"id": 1, "timestamp": "2023-10-04T23:59:59Z", "value": 10},
    {"id": 2, "timestamp": "2023-10-05T00:01:00Z", "value": 20}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Data Source
4. Ingestion Pipeline
5. Is data from latest batch?
6. (Insert into warehouse)
7. Late Arrival Buffer
8. Deduplication Logic
9. Aggregator
10. Warehouse

