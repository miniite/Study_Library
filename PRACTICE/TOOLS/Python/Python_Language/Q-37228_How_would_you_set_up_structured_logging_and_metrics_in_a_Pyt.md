---
type: "practice"
difficulty: "Intermediate"
---

# How would you set up structured logging and metrics in a Python model service so you can answer 'how many predictions per class, at what latency, with what feature-null rate' from a dashboard?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ OpenTelemetry Instrumentation](<../../../../TOOLS/Python/Python_Language/OpenTelemetry_Instrumentation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set up structured logging and metrics in a Python model service, we need to ensure that our system provides detailed insights into prediction frequency, latency, and feature null rates per class. This can be achieved by integrating well-structured loggers and metrics collectors. For this purpose, I would recommend using logging for structured logs and prometheus_client or similar libraries for metrics collection.

#### Triaging the Question

The question is essentially asking how to set up a system that collects and displays detailed performance data from a model service in a dashboard format. The core aspects we need to cover are:

Structured Logging: Ensuring logs contain structured information.
Metrics Collection: Collecting key metrics such as prediction frequency, latency, and feature null rate.
Dashboard Integration: Visualizing the collected data for easy analysis.

#### First-Principles Solution

We will use Python’s logging module to capture structured logs and prometheus_client (or a similar library) for collecting metrics. We'll also set up a Prometheus server or integrate with existing monitoring tools like Grafana for visualization.

Step 1: Setup Structured Logging

First, we need to configure the logging system to output structured logs. This can be done by setting up the logging configuration:

PYTHON
import logging

# Set up basic configuration for logging
logging.basicConfig(level=logging.INFO,
                    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
                    datefmt='%Y-%m-%d %H:%M:%S')

# Create a logger instance
logger = logging.getLogger(__name__)
Step 2: Collect Metrics

Next, we need to define metrics that capture prediction frequency, latency, and feature null rate. We can use prometheus_client for this purpose:

PYTHON
from prometheus_client import Counter, Histogram, start_http_server

# Define counters and histograms for our metrics
predictions_counter = Counter('model_predictions_total', 'Total number of model predictions per class')
latency_histogram = Histogram('model_latency_seconds', 'Response latency in seconds', buckets=[0.1, 0.5, 1.0, 2.0, 3.0, 6.0, float("inf")])
feature_null_rate_gauge = Gauge('feature_null_rate', 'Feature null rate for each prediction')

# Start HTTP server to expose metrics
start_http_server(8000)
Step 3: Instrument Model Service

When the model service makes predictions, it should update these metrics accordingly:

PYTHON
import numpy as np
from time import perf_counter

def make_prediction(input_features):
    # Simulate prediction logic
    start_time = perf_counter()
    prediction_result = np.random.choice(['class_1', 'class_2'])

    # Log structured info and update metrics
    logger.info(f"Prediction for {input_features}, class: {prediction_result}")

    predictions_counter.labels(class=prediction_result).inc()  # Increment counter based on class

    latency_seconds = perf_counter() - start_time
    latency_histogram.observe(latency_seconds)  # Record latency

    null_features_count = sum(np.isnan(input_features))
    feature_null_rate_gauge.set(null_features_count / len(input_features))  # Update feature null rate

    return prediction_result

# Example usage
make_prediction([1.2, np.nan, 3.4])
Step 4: Dashboard Integration

Finally, we need to integrate these metrics with a dashboard such as Grafana:

Grafana: Set up a Prometheus data source and create panels for each metric.
Prometheus Query Language (PQL): Use PQL to write queries for the metrics.

#### Closing Summary

By setting up structured logging and metrics collection using Python’s logging module and prometheus_client, we can effectively monitor the performance of our model service. This setup allows us to track prediction frequency, latency, and feature null rates per class, providing valuable insights through a dashboard.

#### Common Mistakes/Follow-Ups
Not Using Structured Logging: Failing to use structured logging can lead to less actionable logs.
Ignoring Metrics Granularity: Not defining appropriate metrics with sufficient granularity (e.g., per-class prediction counts) can limit the usefulness of collected data.
Dashboard Integration Issues: Incorrect setup in Grafana or Prometheus server configurations can result in missing or incorrect metric visualizations.

