---
type: "practice"
difficulty: "Basic"
---

# How do you decide the schedule (hourly, daily, weekly) for a batch pipeline?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Transform in Pipeline vs Warehouse](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Transform_in_Pipeline_vs_Warehouse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide the schedule for a batch pipeline, we need to consider several factors including data source availability, processing time, required latency, and resource constraints. The key is to strike a balance between getting timely insights and efficient use of resources.

The question asks about scheduling options like hourly, daily, or weekly pipelines. At a basic concept level, let's break down the reasoning from first principles:

Data Source Characteristics: How often does new data become available? If there are real-time updates every minute, an hourly pipeline might be too coarse. Conversely, if data is updated once a day, even daily schedules could be excessive.

Processing Time and Volume: What is the expected processing time for each batch? If it takes several hours to process all incoming data, an hourly schedule could lead to backlogs or slow pipelines. Weekly schedules might miss critical trends in dynamic datasets.

Required Latency: How quickly do you need results? For mission-critical applications like fraud detection, real-time alerts might be necessary, hence hourly updates are preferred. For less time-sensitive tasks, daily summaries can suffice.

Resource Constraints: How many computational resources (CPU, memory) and storage capacity are available? Frequent schedules consume more resources, which could be a limiting factor in resource-limited environments.

For example, consider an e-commerce platform tracking user interactions. If the system gets substantial updates every 15 minutes, processing this data hourly might miss crucial insights. In such cases, setting up daily or weekly pipelines could introduce unacceptable delays.

In summary, we should tailor the pipeline schedule to match the specific needs and characteristics of the project. Frequent schedules (e.g., hourly) ensure timeliness but consume more resources, while infrequent schedules (daily, weekly) reduce resource usage but can increase latency.

**Comparison:**

```
Data Source Interval	Recommended Pipeline
5	hourly
15	hourly
30	hourly
60	daily
240	daily
720	weekly
```

**Code:**

```python
# Example Python code snippet for scheduling a data pipeline
import datetime

def schedule_pipeline(data_source_interval):
    # Assuming the interval is in minutes
    if data_source_interval <= 60:
        return "hourly"
    elif data_source_interval <= 1440:  # 24 hours
        return "daily"
    else:
        return "weekly"

# Example usage with a 30-minute interval
interval = 30
pipeline_schedule = schedule_pipeline(interval)
print(f"Recommended pipeline schedule for {interval} minute intervals:", pipeline_schedule)
```

**Example input / output:**

```text
# Input data source interval in minutes
data_source_interval = 30

# Output recommendation for pipeline schedule
pipeline_schedule = schedule_pipeline(data_source_interval)

print(f"Input: Data Source Interval (minutes) - {data_source_interval}")
print(f"Output: Recommended Pipeline Schedule - {pipeline_schedule}")
```

**Data:**

```json
{
  "data_source_interval": 30,
  "recommended_schedule": "hourly"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Define Data Source Interval
6. Less than 1 hour?
7. Schedule hourly
8. Less than 1 day?
9. Schedule daily
10. Schedule weekly

