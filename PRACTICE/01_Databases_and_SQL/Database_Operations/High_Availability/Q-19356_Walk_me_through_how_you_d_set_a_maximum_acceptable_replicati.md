---
type: "practice"
difficulty: "Deep"
---

# Walk me through how you'd set a maximum acceptable replication lag threshold for automated failover, and what heuristics prevent flapping when the standby is briefly behind.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ HA Failover Design](<../../../../01_Databases_and_SQL/Database_Operations/High_Availability/HA_Failover_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure high availability (HA) in a database environment, setting an appropriate maximum acceptable replication lag threshold for automated failover is crucial. This threshold helps balance between minimizing downtime and avoiding frequent false positives that could lead to flapping. Here’s how you can approach this problem:

#### Overview
Automated failover involves monitoring the primary and secondary databases (standbys) to detect when a standby falls behind the primary. The key challenge is setting a lag threshold that minimizes both unavailability due to actual failures and false positives due to temporary network issues or brief outages.

#### Triage

Question: How do you set an automated failover threshold for replication lag, and what heuristics prevent flapping when the standby is briefly behind?
Core Idea: The solution involves setting a threshold that considers both business requirements and technical constraints.

#### Assumptions
Assume we are using PostgreSQL with streaming replication and have a master (primary) and one or more replicas (standbys).

#### First-Principles Solution

Define the Lag Threshold: The threshold should be based on acceptable downtime for critical data changes and network latency.
For example, if you expect updates to take 10 seconds, a lag of 20 seconds might indicate that something is wrong.
Implement Monitoring: Use tools like pg_stat_replication or custom scripts to monitor the replication status.
Heuristics for Flapping Prevention:
Buffer Time: Introduce a buffer time (e.g., 30 seconds) within which brief lags are considered normal and ignored.
Detection Window: Use a detection window (e.g., 2 minutes) to ensure that only consistent lag issues trigger failover, rather than temporary spikes.
Threshold Calculation: Define the threshold as a multiple of expected network latency. For instance, if network latency is 10 seconds and you expect occasional delays, set the threshold to (3 \times (expected\ latency + buffer)).

Detailed Steps:

Step 1: Estimate Latency: Measure typical network latency between master and standby.
BASH
# Example command on standby
psql -c "SHOW statement_timeout;"
Step 2: Define Buffer Time: Set a buffer time to account for expected delays.
For instance, buffer_time = 30 seconds.
Step 3: Calculate Threshold:
PYTHON
def calculate_threshold(expected_latency, buffer_time):
    return 3 * (expected_latency + buffer_time)

threshold = calculate_threshold(10, 30)  # Example calculation
Step 4: Implement Monitoring: Use PostgreSQL’s pg_stat_replication or a custom script to monitor lag.
SQL
SELECT * FROM pg_stat_replication;
Step 5: Trigger Failover: If the lag exceeds the threshold, initiate failover. This can be done via a cron job or through a database management tool.

#### Closing Summary

Setting an appropriate replication lag threshold involves balancing between acceptable downtime and preventing false positives.
Key steps include defining a buffer time, using a detection window, and calculating the threshold based on expected network latency.
Heuristics like buffer time and detection window help prevent flapping by ignoring brief lags.

#### Common Mistakes/Follow-Ups

Mistake: Ignoring temporary network fluctuations can lead to unnecessary failovers (false positives).

#### Follow-Up
Discuss the importance of continuous monitoring and adjustment of thresholds based on real-world performance data.

**Comparison:**

```
estimateLatency
setBufferTime
calculateThreshold
monitorLag
Setup
Calculation
Monitoring
Calculate Replication Lag Threshold
```

**Code:**

```python
def calculate_threshold(expected_latency, buffer_time):
    return 3 * (expected_latency + buffer_time)

# Example usage
threshold = calculate_threshold(10, 30)
print(f"Replication lag threshold: {threshold} seconds")
```

**Example input / output:**

```text
- **Input**: Expected latency = 10, buffer time = 30.
 - **Output**: Replication lag threshold = 120 seconds.
```

**Data:**

```json
{
  "expected_latency": 10,
  "buffer_time": 30,
  "threshold": 120
}
```

**Flow:**

1. Measure Network Latency
2. Set Buffer Time
3. Calculate Threshold
4. Monitor Replication Lag withpg_stat_replication
5. Check if Lag Exceeds Threshold
6. Initiate Failover
7. Avoid Flapping byImplementing Heuristics

