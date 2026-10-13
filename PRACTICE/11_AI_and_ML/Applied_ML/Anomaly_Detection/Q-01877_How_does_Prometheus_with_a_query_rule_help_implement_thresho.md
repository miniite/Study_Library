---
type: "practice"
difficulty: "Intermediate"
---

# How does Prometheus with a query rule help implement threshold-based anomaly alerting?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Dynamic vs Static Alerting](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Dynamic_vs_Static_Alerting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how Prometheus with a query rule can be used to implement threshold-based anomaly alerting, we need to break down the process step-by-step and understand the core concepts involved.

#### Triage

Question Breakdown: The key here is understanding how Prometheus leverages query rules (also known as Alertmanager alerts) to detect anomalies in time series data. Specifically, we are looking at setting up a threshold-based alerting mechanism where if certain metrics cross predefined thresholds, an alert is triggered.

Interviewer's Probe:

How does Prometheus differentiate between normal and anomalous behavior?
What kinds of thresholds can be used for anomaly detection?
Can you explain the process of configuring these rules in practice?

#### First-Principles Solution

#### Overview

Prometheus uses query rules (Alertmanager alerts) to monitor time series data and trigger alerts when metrics deviate from expected behaviors. The basic idea is to define a set of conditions that, if met, indicate an anomaly.

Step-by-Step Explanation
Define the Metric: Identify the metric you want to monitor for anomalies. For example, CPU usage on a server.
Set Thresholds: Define thresholds that represent normal and abnormal behavior. These can be simple or complex conditions involving multiple metrics and time windows.
Create Query Rules: Use Prometheus rules to evaluate these thresholds over time. Rules are defined in the prometheus.yml configuration file, where you specify matchers for your metric and define how often and under what conditions an alert should be triggered.
Configure Alertmanager: Alertmanager is responsible for sending out notifications when a rule fires. It receives alerts from Prometheus and handles routing based on alert groups.

#### Example
Setup

Suppose we want to monitor the CPU usage of servers, and we suspect that if any server's CPU usage exceeds 80% consistently over the last five minutes, it might be experiencing an anomaly.

Metric Definition: The metric is cpu_usage{host="serverX"}.

Thresholds:

Normal behavior: CPU usage < 75%
Anomaly threshold: CPU usage > 80% consistently over the last 5 minutes

Query Rule Configuration in prometheus.yml:

YAML
# prometheus.yml
rule_groups:
- name: cpu_usage_rules
  rules:
  - alert: HighCPUUsage
    expr: avg by (host) (increase(cpu_usage[5m])) > 80
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "High CPU usage on {{ $labels.host }}"
Alertmanager Configuration:
YAML
# alertmanager.yml
route:
  group_by: ['alertname', 'host']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: email
receivers:
- name: "email"
  email_configs:
  - to: admin@example.com
    from: alertmanager@example.com

#### Closing Summary

By configuring Prometheus query rules, we can effectively monitor time series data and trigger alerts based on predefined thresholds. This approach is crucial for real-time monitoring and proactive issue resolution in AIOps environments.

#### Common Mistakes/Follow-Ups
Threshold Setting: Often, organizations set too high or low thresholds, leading to false positives or negatives.
Complexity of Rules: Overly complex rules can lead to alerts being triggered for noise rather than real anomalies.
Alert Fatigue: Frequent and unhandled alerts can cause users to ignore important alerts over time.

To mitigate these issues, it's essential to continually review and adjust the thresholds based on observed behavior and feedback loops from operations teams.

**Code:**

```yaml
# prometheus.yml example rule group
rule_groups:
- name: cpu_usage_rules
  rules:
  - alert: HighCPUUsage
    expr: avg by (host) (increase(cpu_usage[5m])) > 80
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "High CPU usage on {{ $labels.host }}"
```

