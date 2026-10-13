---
type: "practice"
difficulty: "Deep"
---

# How do you decide between embedding an anomaly-scoring column into lake tables at ingest versus scoring at query time in the serving layer?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Telemetry Lake File Layout](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Telemetry_Lake_File_Layout.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether to embed an anomaly-scoring column directly into the data lake tables during ingestion or to score anomalies at query time, we need to consider several factors that impact observability, performance, and overall system architecture. The core decision hinges on balancing these trade-offs.

Firstly, let's break down what each approach involves:

Ingest-time Scoring: Anomaly scores are calculated and stored directly in the data lake tables when the raw data is ingested.
Query-time Scoring: Anomaly scores are computed only during query execution, leveraging the serving layer for this task.

The key difference lies in where the heavy lifting (computational overhead) occurs. Ingest-time scoring can provide faster response times and reduces the load on the query infrastructure but increases storage costs. Query-time scoring offloads processing to the query layer, potentially improving scalability but at the cost of increased latency and resource consumption during queries.

Now, let's reason through each approach step by step:

Ingest-Time Scoring:

Advantages:
Faster query response times since anomaly scores are already computed and stored.
Easier to maintain a single source of truth for anomaly data.
Disadvantages:
Increased storage requirements due to storing the scoring results alongside raw data.
Higher computational load during ingestion, which might be an issue in high-throughput scenarios.

Query-Time Scoring:

Advantages:
Reduced storage costs since only the raw data is stored, and anomaly scores are computed on-the-fly.
Flexibility to change scoring algorithms or thresholds without reprocessing existing data.
Disadvantages:
Increased latency during queries due to real-time scoring computations.
Potentially higher resource consumption by the serving layer.

To make an informed decision, we should consider several factors:

Scalability and Performance Requirements: If real-time anomaly detection is critical and the data volume is high, ingest-time scoring might be necessary. However, if scalability is a primary concern and the system can tolerate some latency in anomaly responses, query-time scoring could be more suitable.
Resource Constraints: High computational costs during ingestion might necessitate query-time scoring to keep resource usage under control. Conversely, if the data processing infrastructure has ample resources, ingest-time scoring may not pose significant challenges.
Storage Costs: If storage is a critical constraint and ingesting additional anomaly scores would strain the budget, query-time scoring is a more cost-effective option.

In conclusion, the choice between embedding an anomaly-scoring column at ingestion or performing it at query time depends on the specific requirements of the system. Ingest-time scoring offers faster response times and easier maintenance but requires more storage, while query-time scoring reduces storage costs but increases computational overhead during queries. A hybrid approach could also be considered, where a minimal set of precomputed scores is stored alongside raw data with additional computations happening at query time.

**Comparison:**

| Data Storage Approach | Ingest-Time Scoring | Query-Time Scoring |
|-----------------------|--------------------|--------------------|
| Storage Needs         | High                | Low                 |
| Processing Load       | During ingestion    | During queries      |
| Query Response Time   | Fast                | Slower              |
| Cost                  | Higher              | Lower               |

**Code:**

```python
# Example code to illustrate the concept (not executable)
def ingest_and_score_data(data):
    # Ingest and process raw data
    processed_data = preprocess_raw_data(data)

    # Calculate anomaly scores using a scoring function
    for record in processed_data:
        score = calculate_anomaly_score(record)
        record['anomaly_score'] = score

    return processed_data

def query_and_score_data(query):
    # Query the raw data from the data lake
    raw_data = query_raw_data_from_lake(query)

    # Compute anomaly scores on-the-fly during query execution
    for record in raw_data:
        score = calculate_anomaly_score(record, real_time=True)
        record['anomaly_score'] = score

    return raw_data
```

**Example input / output:**

```text
Input:
Ingest the raw data and compute anomaly scores.

Output (Ingest-time Scoring):
[
    { "timestamp": "2023-10-01T00:00Z", "value": 10, "category": "A", "anomaly_score": 0.9 },
    { "timestamp": "2023-10-01T01:00Z", "value": 15, "category": "B", "anomaly_score": 0.8 },
    { "timestamp": "2023-10-01T02:00Z", "value": 20, "category": "A", "anomaly_score": 0.7 }
]

Output (Query-time Scoring):
[
    { "timestamp": "2023-10-01T00:00Z", "value": 10, "category": "A", "anomaly_score": 0.9 },
    { "timestamp": "2023-10-01T01:00Z", "value": 15, "category": "B", "anomaly_score": 0.8 },
    { "timestamp": "2023-10-01T02:00Z", "value": 20, "category": "A", "anomaly_score": 0.7 }
]
```

**Data:**

```json
{
  "raw_data": [
    { "timestamp": "2023-10-01T00:00Z", "value": 10, "category": "A" },
    { "timestamp": "2023-10-01T01:00Z", "value": 15, "category": "B" },
    { "timestamp": "2023-10-01T02:00Z", "value": 20, "category": "A" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Ingest Data
4. Is Ingest-Time ScoringNeeded?
5. Ingest and Score
6. Query Raw Data
7. Calculate Anomaly ScoresDuring Query
8. Store Ingested Data withScores
9. Return Annotated Results

