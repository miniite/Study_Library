---
type: "practice"
difficulty: "Deep"
---

# An interviewer challenges: 'Your whole streaming design assumes Kafka is up. What's your plan when Kafka itself is the incident?' Reason through detection and fallback when the backbone degrades.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Consuming Untrusted Streams](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Consuming_Untrusted_Streams.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a real-time streaming architecture, it’s crucial to anticipate failures at all levels, including the infrastructure that forms its backbone—such as Kafka. If we rely on Kafka for critical data processing and it goes down, our system must have robust fallback mechanisms in place to ensure continuous operation. This involves monitoring Kafka's health, implementing failover strategies, and ensuring alternative data storage or processing options are available.

Firstly, let’s break down the problem: if Kafka fails, what happens? We need to detect this failure early and react quickly. This typically involves setting up a monitoring system that can continuously check for Kafka’s availability and performance metrics such as lag, connectivity, and resource utilization.

#### Triage
What is Being Probed

The interviewer is probing our understanding of:

Monitoring and Alerting: How we detect when the Kafka cluster is not functioning correctly.
Fault Tolerance: What plans exist to ensure that processing can continue even if Kafka fails.
Recovery Strategies: Steps taken to bring back the system or switch to an alternative solution.

#### Assumptions
Kafka Clusters: We assume there are multiple Kafka clusters for high availability and load distribution.
Monitoring Infrastructure: A reliable monitoring infrastructure is in place, capable of handling alerts and providing insights into system health.
Fallback Mechanisms: Alternative data storage or processing systems are available to handle the workload if Kafka fails.
Step-by-Step Solution

Detection Mechanism:

Health Checks: Implement health checks using tools like Prometheus or Grafana to continuously monitor Kafka’s availability and performance metrics.
Proactive Alerts: Set up alerting on these metrics, ensuring that operations teams are notified if any critical thresholds are breached.

Fallback Strategies:

Replica Recovery: If a Kafka broker fails, ensure its replica can take over quickly to maintain service continuity.
Data Replication: Consider using redundant data storage solutions such as another message queue system (e.g., AWS Kinesis) or distributed file systems (e.g., S3).
Load Balancing: Diversify the input sources and processing logic so that if one Kafka cluster fails, other clusters can still handle the load.

Alternative Processing:

Hybrid Architecture: Design a hybrid architecture where critical components of the pipeline can switch to alternative data pipelines or storage solutions.
Manual Failover: Develop scripts for manual failover in case automated mechanisms are insufficient. Ensure that these scripts are tested and documented thoroughly.
Automated Recovery: Implement automation for recovering from failures, such as using Kubernetes to automatically redeploy containers when they crash.

#### Closing Summary

In summary, the key is a proactive approach to monitoring Kafka’s health with detailed alerts and failover strategies in place. This ensures that when Kafka faces issues, the system can quickly switch to alternative processing or storage mechanisms without downtime. The architecture should be designed with redundancy and resilience at its core, ensuring high availability even under adverse conditions.

#### Common Mistakes/Follow-Ups
Overreliance on Single Point of Failure: Assuming a single Kafka cluster is sufficient.
Lack of Monitoring Tools: Failing to set up comprehensive monitoring for early detection of issues.
Inadequate Testing: Not thoroughly testing failover mechanisms in real-world scenarios.

**Comparison:**

| Metric | Thresholds        | Action                              |
|--------|------------------|------------------------------------|
| Lag    | > 100 messages    | Scale up Kafka brokers             |
| CPU    | > 90% utilization | Add more resources to Kafka nodes  |
| Network | < 50 Mbps     | Upgrade network infrastructure      |

**Code:**

```python
# Example of health check and alerting setup using Grafana
grafana_dashboard = """
panel_id: 1
title: Kafka Broker Health Check
time_series:
- target: rate(node_exporter_kafka_broker_down{job="kafka", instance=~".*"}[5m])
  alert: Kafka Broker Down
  annotations:
    summary: "Kafka broker down detected. Alerting."
"""
```

**Example input / output:**

```text
# Dry-run for detecting and switching to alternative storage
Input: Kafka broker 'broker1' failed with error code 2 (Timeout).

Detection Output:
- Kafka broker down alert sent to ops team.
- Alternative data source 'AWS Kinesis' detected as active.

Fallback Strategy Output:
- Data processing switched from Kafka to AWS Kinesis.
```

**Data:**

```yaml
# Sample configuration for health checks and alerts
monitoring_config:
  kafka_broker_down_threshold: 30s # Time in seconds before an alert is triggered if no response from Kafka brokers.
  kafka_health_check_interval: 5m # Interval at which the health of Kafka brokers is checked.
```

**Flow:**

1. Yes
2. No
3. Health Check
4. Kafka Up?
5. Continue Processing
6. Switch to Fallback Mechanism
7. Monitor Fallback Mechanism
8. Recover Kafka

