---
type: "practice"
difficulty: "Intermediate"
---

# What is the value of combining traces with metric anomalies during incident investigation?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Multivariate Anomaly Correlation](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Multivariate_Anomaly_Correlation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In incident investigation, particularly when dealing with complex systems like those managed by AIOps, the combination of trace analysis and anomaly detection in time series metrics can significantly enhance our ability to pinpoint issues quickly and accurately. This approach leverages the strengths of both methods—traces for detailed contextual insights and anomaly detection for identifying unusual patterns in data—to provide a holistic view that aids in faster troubleshooting.

The question is essentially probing how combining these two techniques can improve incident investigation efficiency and effectiveness. We need to assume that we have access to both trace data (representing the flow of requests within an application) and time series metrics (such as request latency, response times, etc.). The core idea here is to understand how these methods complement each other.

Let's break this down from first principles:

Understanding Traces: Traces provide a detailed view of what happened during a particular incident by tracking the sequence of function calls and their interactions. Each trace can capture the journey of an individual request, including where it started, which functions were called, and how long each took.

Anomaly Detection in Time Series Metrics: Anomaly detection involves identifying unusual patterns that deviate from expected behavior in time series data. This could be anything from sudden spikes in latency to unexpected drops in throughput.

Combining Traces with Metric Anomalies:

Identifying the Problem Area: By combining trace data with metric anomalies, we can pinpoint specific parts of a system where issues are occurring more frequently or have a larger impact.
Contextualizing Metrics: A single metric anomaly might not be enough to determine the root cause. However, when paired with relevant traces, it provides context about which requests were affected and how they interacted within the system.
Reducing False Positives/Negatives: Traces help in verifying whether an observed anomaly is real or a false positive, while metrics provide quantitative evidence of abnormal behavior.

Here’s a step-by-step reasoning to fully address this:

Collect Trace Data and Metrics:

PYTHON
# Example code snippet for collecting traces and metrics
from opentracing.tracer import Tracer

def collect_data():
    tracer = Tracer()
    trace_id, span_context = tracer.start_trace('incident')
    response_times = []  # Time series metric collection
    requests = []        # Trace data collection

    while some_condition:
        response_time = measure_response_time()  # Simulate measurement
        response_times.append(response_time)

        with tracer.new_span(operation_name='request'):
            request_details = make_request()
            requests.append(request_details)

    return response_times, requests

Analyze Metrics for Anomalies:

PYTHON
from statsmodels.tsa.stattools import acf

def detect_anomalies(response_times):
    # Use statistical methods to identify anomalies
    autocorrelation = acf(response_times, fft=True)
    threshold = 3 * np.std(autocorrelation)  # Adjust threshold based on standard deviation
    for i, value in enumerate(acf(response_times)):
        if abs(value) > threshold:
            print(f"Anomaly detected at time index: {i}")

Correlate Anomalies with Traces:

PYTHON
def correlate_traces_with_anomalies(anomalies, requests):
    for anomaly in anomalies:
        for request in requests:
            if request['start_time'] <= anomaly < (request['end_time'] + 10):  # Adjust time window
                print(f"Trace {request} is associated with an anomaly at index: {anomaly}")

By combining these steps, we can effectively identify and investigate anomalies that impact system performance, leveraging both the detailed context provided by traces and the quantitative insights from metric anomalies.

**Code:**

```python
from opentracing.tracer import Tracer

def collect_data():
    tracer = Tracer()
    trace_id, span_context = tracer.start_trace('incident')
    response_times = []  # Time series metric collection
    requests = []        # Trace data collection

    while some_condition:
        response_time = measure_response_time()  # Simulate measurement
        response_times.append(response_time)

        with tracer.new_span(operation_name='request'):
            request_details = make_request()
            requests.append(request_details)

    return response_times, requests

def detect_anomalies(response_times):
    from statsmodels.tsa.stattools import acf

    autocorrelation = acf(response_times, fft=True)
    threshold = 3 * np.std(autocorrelation)  # Adjust threshold based on standard deviation
    for i, value in enumerate(acf(response_times)):
        if abs(value) > threshold:
            print(f"Anomaly detected at time index: {i}")

def correlate_traces_with_anomalies(anomalies, requests):
    for anomaly in anomalies:
        for request in requests:
            if request['start_time'] <= anomaly < (request['end_time'] + 10):  # Adjust time window
                print(f"Trace {request} is associated with an anomaly at index: {anomaly}")
```

**Example input / output:**

```text
# Example input and output dry-run

response_times, requests = collect_data()

detect_anomalies(response_times)

correlate_traces_with_anomalies(anomalies=[30, 50, 70], requests=requests)
```

**Flow:**

1. Yes
2. No
3. Start
4. Are there anomalies in themetrics?
5. Determine if traces correlatewith anomalies
6. Analyze and investigatecorrelated issues
7. No significant anomaliesdetected, continue monitoring
8. End

