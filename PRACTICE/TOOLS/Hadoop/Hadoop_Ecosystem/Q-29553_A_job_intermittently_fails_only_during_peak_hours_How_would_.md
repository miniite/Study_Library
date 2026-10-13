---
type: "practice"
difficulty: "Deep"
---

# A job intermittently fails only during peak hours. How would you correlate YARN resource contention, node health, and container preemption to find the cause?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ YARN Troubleshooting](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/YARN_Troubleshooting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose intermittent Hadoop YARN job failures during peak hours, we need to methodically analyze several factors related to resource contention, node health, and container preemption. This involves understanding how these components interact within a YARN cluster. Let's break down the problem from first principles.

First, YARN resource contention refers to situations where demand for resources (CPU, memory) exceeds available supply during peak times. Node health encompasses issues like node failures or poor performance due to hardware constraints or maintenance activities. Container preemption involves reclaiming resources from underutilized containers when necessary, potentially leading to job disruptions.

The goal is to correlate these factors to pinpoint the root cause of the failures.

**Comparison:**

| NodeID | Failure Type     | Last Seen        |
|--------|------------------|------------------|
| n001   | Disk space low   | 2023-04-01T15:00Z|
| n007   | Network issue    | 2023-04-02T18:00Z|

**Example input / output:**

```text
| NodeID | CPU Usage (%) | Memory Usage (%) |
|--------|--------------|-----------------|
| n001   | 85           | 70              |
| n002   | 79           | 65              |
| ...    | ...          | ...             |

Node failure logs:
```

**Data:**

```json
{
  "resourceUsage": {
    "peakHourCPUUsage": 80,
    "peakHourMemoryUsage": 75,
    "totalNodes": 100,
    "failedJobsDuringPeak": [
      {"startTime": "2023-04-01T12:00Z", "endTime": "2023-04-01T13:00Z"},
      {"startTime": "2023-04-02T16:00Z", "endTime": "2023-04-02T17:00Z"}
    ]
  },
  "nodeHealthIssues": [
    {"nodeID": "n001", "issue": "Disk space low"},
    {"nodeID": "n056", "issue": "Network connectivity issues"}
  ],
  "containerPreemptionEvents": {
    "preemptedContainers": 20,
    "reclaimedResources": "3GB RAM, 4CPU"
  }
}
```

**Flow:**

1. Correlation
2. Resource Contention
3. Node Health
4. Container Preemption
5. Finding Root Cause

