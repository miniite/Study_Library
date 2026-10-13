---
type: "practice"
difficulty: "Deep"
---

# Your Python log-anomaly pipeline that clusters log templates falls behind during an incident when log volume spikes 20x. How would you add backpressure so it degrades to sampling instead of OOMing?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Memory Leaks in Batch Jobs](<../../../../TOOLS/Python/Python_Language/Memory_Leaks_in_Batch_Jobs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where a Python-based log-anomaly pipeline struggles under high load, leading to out-of-memory (OOM) errors, we need to add backpressure mechanisms that allow the system to degrade gracefully by resorting to sampling instead of processing every single log. This requires understanding how backpressure works in systems and implementing it within our logging pipeline.

#### Triage

The core question here is: How can we design a Python-based log-anomaly detection pipeline to handle increased load without crashing due to OOM errors? The interviewer is likely probing your knowledge of system design, error handling, and resource management. We will make several key assumptions:

The current pipeline processes logs sequentially in real-time.
Logs are ingested from a high-velocity source like a network interface or log file.
The anomaly detection logic runs on a single thread or a limited number of threads.

#### Solution

From first principles, we need to introduce mechanisms that can dynamically adjust the rate at which logs are processed based on system resource availability. This involves:

Monitoring System Resources: Continuously monitoring CPU usage and memory consumption.
Dynamically Adjusting Processing Rate: Slowing down log processing when resources are under pressure.
Using Sampling Techniques: Implementing a sampling strategy to process only a subset of logs during peak times.
Step-by-Step Solution

Monitoring System Resources

Use system monitoring libraries like psutil or operating system APIs to monitor CPU and memory usage in real-time.
Define thresholds for when the system is considered under high load (e.g., 80%+ CPU utilization, 75%+ memory usage).

Dynamically Adjusting Processing Rate

Introduce a backpressure mechanism that limits the number of logs processed per second.
Use a token bucket algorithm to control the flow of logs into the anomaly detection pipeline.

Using Sampling Techniques

Implement log sampling logic where, during high load periods, only a fraction of incoming logs are processed for anomaly detection.
For example, if we detect that CPU usage is above 80%, instead of processing all logs, we process every nth log (e.g., every 10th or 50th log).
Code Example
PYTHON
import psutil
from datetime import datetime

# Constants for threshold and sampling rate
CPU_THRESHOLD = 0.8
MEMORY_THRESHOLD = 0.75
SAMPLING_INTERVAL = 10  # Process only every nth log

def process_logs(logs, pipeline):
    now = datetime.now()
    cpu_usage = psutil.cpu_percent(interval=1)
    memory_usage = psutil.virtual_memory().percent

    if cpu_usage < CPU_THRESHOLD and memory_usage < MEMORY_THRESHOLD:
        for log in logs:
            result = pipeline.analyze_log(log)
            print(result)
    else:
        # Implement sampling logic
        for i, log in enumerate(logs):
            if (i + 1) % SAMPLING_INTERVAL == 0:  # Process every nth log
                result = pipeline.analyze_log(log)
                print(f"Sampling {log}: {result}")

#### Closing Summary
and Common Mistakes

By implementing backpressure mechanisms such as dynamic rate limiting and sampling, we ensure that the Python log-anomaly detection pipeline can handle increased load without crashing. The key is to continuously monitor system resources and adjust processing rates accordingly. Common mistakes include not monitoring resources effectively, not adjusting processing rates dynamically, and failing to implement sampling logic which can lead to unnecessary resource consumption.

