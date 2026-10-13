---
type: "practice"
difficulty: "Deep"
---

# How do you design a graceful-degradation path so that when the feature pipeline is late the anomaly detector falls back to last-good baselines with an explicit staleness contract?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Shared Pipeline Platforms](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing/Shared_Pipeline_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure robustness in our AIOps system, we need a mechanism where the anomaly detector can gracefully degrade its performance when the feature pipeline is late or unavailable. This solution involves designing a fallback strategy that relies on last-good baselines and includes an explicit staleness contract to clearly define how old data can be before it's considered stale.

In essence, this question probes your understanding of error handling, robust system design, and the trade-offs between accuracy and availability in real-time systems. We'll assume the primary feature pipeline uses Apache Airflow for orchestration and Apache Spark for processing, with a secondary baseline source that stores historical data.

#### Triage

The core challenge here is to ensure that the anomaly detector can operate effectively even when the main feature pipeline encounters delays or failures without compromising too much on accuracy. This involves:

Defining a staleness contract: How old can the last-good baseline be before it's considered insufficient?
Implementing fallback logic in the anomaly detection algorithm.
Ensuring both the primary and secondary data sources are reliable and well-integrated.

#### Solution
Step 1: Define Staleness Contract

First, we need to establish a clear staleness contract. For instance, let’s assume that any data older than 5 minutes should be considered stale if it cannot be fetched from the feature pipeline within this timeframe. This threshold can be adjusted based on real-world requirements and the criticality of the anomaly detection results.

Step 2: Integrate Last-Good Baselines

Next, we need to integrate last-good baselines into our system. These could be stored in a reliable secondary data source such as an S3 bucket or a time-series database like InfluxDB. The key is to ensure that these baseline values are updated regularly and are accessible even when the primary pipeline fails.

Step 3: Implement Fallback Logic

The anomaly detection algorithm must include fallback logic. This can be achieved by:

Primary Pipeline Check: First, attempt to fetch the latest data from the Airflow/Apache Spark feature pipeline.
Fallback to Baseline: If the primary pipeline fails (e.g., exceeds the staleness threshold), switch to the last-known good baseline stored in the secondary data source.
Staleness Notification: Log a notification that the fallback has occurred, along with the time of the failure and the age of the last-good baseline.
Step 4: Monitoring and Logging

Implement comprehensive monitoring and logging to track when the anomaly detector falls back to baselines. This will help in diagnosing issues quickly and understanding the system's behavior under stress scenarios.

#### Example
Code Snippet for Fallback Logic
PYTHON
def detect_anomalies(data_pipeline):
    try:
        # Attempt to fetch data from the primary pipeline
        features = data_pipeline.fetch_features()
    except DataFetchTimeoutError as e:
        # Log a warning and use fallback logic
        logger.warning(f"Failed to fetch features: {e}")

        # Check staleness of last-good baseline
        if is_stale(last_good_baseline_timestamp, current_time):
            raise BaselineStaleError("Baseline data too old")

        # Use last-known good baseline
        return use_last_good_baseline()

    return analyze_features(features)

def is_stale(timestamp, current_time, staleness_threshold=300):  # 5 minutes in seconds
    return (current_time - timestamp).total_seconds() > staleness_threshold

def use_last_good_baseline():
    # Retrieve last good baseline from secondary data source
    baseline_data = fetch_last_good_baseline()
    return analyze_features(baseline_data)

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, the solution involves defining a clear staleness contract, integrating reliable last-good baselines, implementing robust fallback logic, and ensuring comprehensive monitoring. Common mistakes include not setting appropriate thresholds for staleness, failing to properly update baseline data, or neglecting thorough logging of fallback events.

For follow-ups, we could explore:

How to improve the reliability of secondary data sources.
Strategies for reducing the latency in fetching primary pipeline data.
Incorporating machine learning models that can predict potential failures and optimize data caching.

**Comparison:**

```
Staleness Contract	Description
BaselineStaleError	Thrown if the baseline data is too old
DataFetchTimeout	Indicates failure to retrieve data from primary pipeline within timeout period
```

**Code:**

```python
def detect_anomalies(data_pipeline):
    try:
        features = data_pipeline.fetch_features()
    except DataFetchTimeoutError as e:
        logger.warning(f"Failed to fetch features: {e}")

        if is_stale(last_good_baseline_timestamp, current_time):
            raise BaselineStaleError("Baseline data too old")

        return use_last_good_baseline()

    return analyze_features(features)

def is_stale(timestamp, current_time, staleness_threshold=300):
    return (current_time - timestamp).total_seconds() > staleness_threshold

def use_last_good_baseline():
    baseline_data = fetch_last_good_baseline()
    return analyze_features(baseline_data)
```

**Example input / output:**

```text
Input:
- `data_pipeline.fetch_features()` returns a timeout error.
Output:
- Fallback to last-good baseline data `[0.5, 0.8, 0.6]`.
- Log entry: "Failed to fetch features: Timeout exceeded; falling back to last-known good baseline."
```

**Data:**

```json
{
  "baseline_timestamp": "2023-10-01T14:00:00Z",
  "last_good_baseline": [0.5, 0.8, 0.6]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Detect Anomalies
6. Is Primary Pipeline Available?
7. Fetch Features
8. Analyze Features
9. Is Baseline Stale?
10. Throw BaselineStaleError
11. Use Last-Good Baseline
12. Log Error and Fall Back

