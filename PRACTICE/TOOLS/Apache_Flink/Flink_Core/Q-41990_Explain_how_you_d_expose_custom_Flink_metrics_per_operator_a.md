---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd expose custom Flink metrics per operator and wire them to Prometheus for the anomaly pipeline's own SLOs.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Keyed vs Operator State](<../../../../TOOLS/Apache_Flink/Flink_Core/Keyed_vs_Operator_State.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain how to expose custom Flink metrics per operator and wire them to Prometheus for an anomaly detection pipeline’s Service Level Objectives (SLOs), we need a clear understanding of both Flink's streaming capabilities and Prometheus' metric collection and visualization features. This involves several steps, from setting up the necessary configurations in Flink to defining proper Prometheus scraping intervals.

TL;DR
Configure Metrics in Flink: Use MetricGroup or MetricRegistry to define custom metrics per operator.
Expose Metrics via REST API: Enable Flink’s REST endpoint for metrics and expose them over HTTP.
Set Up Prometheus Scrape Job: Configure Prometheus to scrape the Flink metrics using a job with appropriate labels and paths.
Define SLOs in Prometheus: Use these metrics to define rules for anomaly detection, ensuring the pipeline meets its SLOs.

#### Triaging the Question

The core of this question revolves around understanding how to integrate custom metrics from Apache Flink into a monitoring system like Prometheus. The interviewer is probing your knowledge of:

Flink Metrics: How to create and expose them.
Prometheus Monitoring: Configuring scraping jobs, setting up alerts, and defining SLOs.
Integration Best Practices: Ensuring smooth data flow between the two systems.

#### Assumptions
You have a basic understanding of Flink’s streaming architecture.
Prometheus is already running and configured to scrape metrics from other services.
The anomaly pipeline has defined SLAs or SLOs that need monitoring.
Step-by-Step Solution
1. Configuring Metrics in Flink

Apache Flink provides MetricGroup for creating custom metrics per operator. Here’s a simple example:

JAVA
final MetricGroup metricGroup = getRuntimeContext().getMetricGroup();
metricGroup.gauge("custom_operator_metric", () -> getCurrentValue(), null);

This snippet shows how to create a gauge metric called "custom_operator_metric" that returns the current value of some state or operation within an Flink job.

2. Exposing Metrics via REST API

To expose these metrics, you can enable the Flink web UI and use its built-in REST endpoint for metrics:

PROPERTIES
# In your flink-conf.yaml
rest.server.address=localhost
rest.server.port=8091

Ensure that the metrics.reporter.prometheus is configured correctly in flink-conf.yaml, which enables Prometheus scraping on a specific HTTP port.

3. Setting Up Prometheus Scrape Job

Prometheus needs to be configured with a job that scrapes metrics from Flink’s REST API endpoint. Add this configuration to your prometheus.yml:

YAML
scrape_configs:
  - job_name: 'flink-metrics'
    static_configs:
      - targets: ['localhost:8091/metrics']
        labels:
          component: 'flink'
          environment: 'production'

Ensure that the port and path are correctly specified to match Flink’s setup.

4. Defining SLOs in Prometheus

With metrics exposed, you can define SLOs using Prometheus rules or alerts. For example, if your anomaly pipeline has a requirement to ensure at least 95% of requests are processed within 100ms:

YAML
rule_file: 'slo-rules.yml'

In slo-rules.yml, you might have something like this:

YAML
groups:
  - name: flink-slos
    rules:
      - alert: FlinkAnomalyDetection
        expr: rate(custom_operator_metric[5m]) < 0.95
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: 'Flink Anomaly Detected'
          description: 'The custom metric is below the desired SLO of 95% within the last 5 minutes.'

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Forgetting to enable Flink’s metrics reporter or misconfiguring it.

#### Follow-Up
Discussing how to handle different types of custom metrics (e.g., histograms, counters) in Prometheus.
Discussion Point: The importance of proper label usage in Prometheus for better monitoring and alerting.

**Code:**

```yaml
# Example prometheus.yml configuration snippet
scrape_configs:
  - job_name: 'flink-metrics'
    static_configs:
      - targets: ['localhost:8091/metrics']
        labels:
          component: 'flink'
          environment: 'production'
```

