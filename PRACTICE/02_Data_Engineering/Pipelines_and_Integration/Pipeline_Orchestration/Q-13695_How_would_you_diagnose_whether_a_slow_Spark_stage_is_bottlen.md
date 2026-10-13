---
type: "practice"
difficulty: "Deep"
---

# How would you diagnose whether a slow Spark stage is bottlenecked on shuffle read GC pause or executor CPU using only the Spark UI and event logs?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Profiling Spark Stages](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Profiling_Spark_Stages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose whether a slow Spark stage is bottlenecked by shuffle read GC pauses or executor CPU, we'll need to conduct an in-depth analysis leveraging both the Spark UI and event logs. The key idea here is to systematically gather insights from these sources and reason through potential bottlenecks step-by-step.

Firstly, it's crucial to understand that slow stages can be influenced by multiple factors such as network latency, memory management (GC pauses), or CPU limitations. By examining various metrics in the Spark UI and analyzing event logs, we can pinpoint where the slowdown occurs.

**Comparison:**

| Executor ID | Shuffles Read Time (ms) | CPU Time (ms) |
|-------------|------------------------|--------------|
| node-1      | 200                    | 300          |
| node-2      | 250                    | 350          |

**Code:**

```python
# Sample Python code for basic stage analysis using Spark UI API
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("DiagnoseSlowStage").getOrCreate()

def get_stage_details(stage_id):
    # Fetch detailed stage information from the Spark UI API
    response = spark._sc.statusTracker().getStageInfo(stage_id)
    return response['details']

stage_id = 123456  # Example stage ID
stage_details = get_stage_details(stage_id)
print(stage_details)
```

**Example input / output:**

```

Consider a slow stage with an ID of 123456. The get_stage_details function fetches the stage's details from the Spark UI API. For instance:

JSON
Copy
{
  "details": {
    "tasks": [
      {"id": 0, "result": {"durationMs": 100}},
      ...
    ],
    "taskMetrics": [
      {"executorId": "node-1", "metrics": {"shufflesReadTimeMs": 200, "cpuTimeMs": 300}},
      {"executorId": "node-2", "metrics": {"shufflesReadTimeMs": 250, "cpuTimeMs": 350}},
      ...
    ]
  }
}

By analyzing the taskMetrics section, we can identify which tasks are experiencing high shuffle read times or CPU usage.
```

**Data:**

```json
{
  "id": 123456,
  "attemptId": "abc-def-ghi",
  "name": "Stage 0: MapPartitionsRDD[7] at mapPartitions in <function1>",
  "submissionTime": 1689234567,
  "completionTime": 1689234568,
  "status": "FAILED",
  "details": {
    "tasks": [
      {"id": 0, "result": {"durationMs": 100}},
      {"id": 1, "result": {"durationMs": 150}},
      ...
    ],
    "taskMetrics": [
      {"executorId": "node-1", "metrics": {"shufflesReadTimeMs": 200, "cpuTimeMs": 300}},
      {"executorId": "node-2", "metrics": {"shufflesReadTimeMs": 250, "cpuTimeMs": 350}},
      ...
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is the 'shufflesReadTimeMs'high for any task?
7. Check if multiple executors areexperiencing GC pauses
8. Investigate GC logs or usespark.ui.showTaskFailedDetails=true to see if GC is causingslowness
9. Is the 'cpuTimeMs' high forany task?
10. Analyze detailed executor CPUmetrics
11. Look into potential CPUbottlenecks like heavy I/O,network latency
12. No significant GC or CPUissues; consider other factorslike network, JVM parameters

