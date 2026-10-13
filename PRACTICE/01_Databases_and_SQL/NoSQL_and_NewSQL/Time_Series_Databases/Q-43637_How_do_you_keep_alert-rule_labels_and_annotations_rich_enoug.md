---
type: "practice"
difficulty: "Intermediate"
---

# How do you keep alert-rule labels and annotations rich enough for the automation engine to act, without exploding notification cardinality?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Prometheus TSDB and Scrape Health](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Prometheus_TSDB_and_Scrape_Health.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how to maintain a balance between keeping alert-rule labels and annotations rich enough for an automation engine to effectively act while avoiding excessive notification cardinality, we need to consider several key aspects of time-series databases (TSDBs) such as Prometheus or InfluxDB. The core idea is to design alert rules in a way that leverages structured metadata efficiently without overloading the system with redundant notifications.

The question here is essentially asking for a strategy to manage alert complexity while ensuring enough context for automated responses. This involves balancing between alert rule richness and notification volume, which can be complex due to potential combinatorial explosion of alerts based on multiple labels and annotations.

#### Triaging the Question
What are Alert Rules and Labels? Alert rules define conditions under which an alarm is triggered. Labels provide metadata about the metric being monitored, helping in identifying specific instances.
Notifications Cardinality: This refers to the total number of possible combinations of alerts that can be generated based on various label values. High cardinality means a large number of potential alert permutations.
Automation Engine Requirements: The automation engine needs sufficient context from labels and annotations to make informed decisions, such as applying corrective actions or triaging issues.

#### First-Principles Solution

To solve this problem, we can follow these steps:

Define Essential Labels and Annotations: Identify the minimum set of labels needed for each alert rule to function correctly.
Use Logical Grouping: Organize similar alerts into logical groups that share common contexts.
Dynamic Label Selection: Implement a dynamic mechanism to select relevant labels based on real-time conditions or predefined thresholds.
Threshold Management: Set appropriate cardinality limits to prevent excessive notifications while ensuring necessary context is maintained.
Detailed Implementation

Identify Essential Labels:

For instance, in a network monitoring scenario, essential labels might include host, service, and severity.

Logical Grouping:

Group alerts by service type or criticality to reduce the total number of unique alert permutations.

#### Example
YAML
# Alert rule for high CPU usage in production services
groups:
  - name: "cpu_high"
    rules:
      - alert: HighCPULoad
        expr: rate(node_cpu_seconds_total{job="prometheus", mode="user"}[1m]) > 0.8
        labels:
          severity: critical
          service: production
        annotations:
          summary: "High CPU usage detected in {{ $labels.service }}"

Dynamic Label Selection:

Use custom logic to dynamically select labels based on the current state or context.

#### Example
PYTHON
def dynamic_label_selector(context):
    if is_production_service(context):
        return {'severity': 'critical', 'service': 'production'}
    else:
        return {'severity': 'warning', 'service': 'non-production'}

Threshold Management:

Set limits on the number of unique alert combinations to prevent cardinality explosion.

#### Example
threshold settings in Prometheus configuration:
YAML
global:
  evaluate_interval: 15s
  scrape_interval: 15s
  max_shielded_series_per_group: 2000

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, the key is to balance alert rule richness with notification volume by carefully selecting essential labels, using logical grouping, implementing dynamic label selection, and managing cardinality thresholds. Common pitfalls include overloading the system with too many labels or not providing enough context for automated actions.

For follow-ups, consider discussing how different data retention policies can impact alerting strategies or exploring advanced techniques like rule cascades in Prometheus to further optimize alert management.

**Code:**

```python
def dynamic_label_selector(context):
    if is_production_service(context):
        return {'severity': 'critical', 'service': 'production'}
    else:
        return {'severity': 'warning', 'service': 'non-production'}

# Example usage in a Prometheus rule
rule = {
    "alert": "HighCPULoad",
    "expr": "rate(node_cpu_seconds_total{job=\"prometheus\", mode=\"user\"}[1m]) > 0.8",
    "labels": dynamic_label_selector(context),
    "annotations": {"summary": "High CPU usage detected in {{ $labels.service }}"}
}
```

