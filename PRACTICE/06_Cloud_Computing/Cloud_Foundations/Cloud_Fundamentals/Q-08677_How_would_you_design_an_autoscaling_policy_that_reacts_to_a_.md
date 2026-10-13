---
type: "practice"
difficulty: "Deep"
---

# How would you design an autoscaling policy that reacts to a custom business metric like orders-per-minute instead of CPU, and what publishing latency issues would you watch for?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Autoscaling Metrics](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Autoscaling_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an autoscaling policy based on a custom business metric such as orders-per-minute (OPM), we need to focus on real-time data processing and ensure our system can handle the dynamic changes in load efficiently. This involves setting up a pipeline that collects, processes, and publishes OPM data, which then triggers scaling actions. The key challenge is managing publishing latency—ensuring timely updates so that the autoscaler can respond quickly to changing conditions.

#### Triage
What's Being Probed

The interviewer wants you to demonstrate your understanding of cloud fundamentals, particularly in designing real-time systems and handling asynchronous data processing. They are probing for knowledge on:

How to set up a scalable and reliable infrastructure.
Techniques for efficient data collection and processing.
Strategies to minimize publishing latency.
Methods for triggering autoscaling actions based on custom metrics.

#### First-Principles Solution
Data Collection:
Use cloud-native services like CloudWatch or Prometheus to monitor the OPM metric.
Data Processing:
Implement a real-time stream processing pipeline using frameworks such as Apache Kafka, AWS Kinesis, or Google Pub/Sub.
Publishing and Latency Management:
Ensure low-latency data delivery by optimizing network configurations and using reliable message brokers.
Autoscaling Logic:
Configure the autoscaler (e.g., Auto Scaling Groups in EC2) to scale based on the OPM metric, setting thresholds for scaling up or down.
Step-by-Step Solution

Set Up Data Collection:

Deploy CloudWatch metrics or Prometheus exporters to track OPM.
PYTHON
# Example Python script using Prometheus client library
from prometheus_client import start_http_server, Counter

order_counter = Counter('orders_per_minute', 'Number of orders per minute')

def process_order(order):
    order_counter.inc()
    # Process the order...

Stream Processing Pipeline:

Use Apache Kafka to ingest and process OPM data in real-time.
BASH
# Example Kafka consumer setup
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic orders-topic

Publishing Data:

Ensure reliable message delivery by configuring retries and using batch processing to reduce the number of messages sent over the network.
JSON
// Kafka producer configuration with retries
{
  "retries": 3,
  "batch.size": 16384
}

Autoscaling Configuration:

Configure Auto Scaling policies in AWS to scale based on the OPM metric from CloudWatch.
YAML
# Example AWS Auto Scaling Group configuration
---
LaunchTemplate:
  Type: "AWS::AutoScaling::LaunchConfiguration"
  Properties:
    ImageId: "ami-1234567890abcdef0"
    InstanceType: "t3.micro"
    UserData:
      Fn::Base64: !Sub |
        #!/bin/bash
        echo "Start scaling based on OPM..."

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By setting up a real-time stream processing pipeline, we can ensure that the autoscaler receives timely updates about changes in OPM. This requires careful consideration of publishing latency to maintain responsiveness.

#### Common Mistakes
Ignoring network delays or message broker latencies, leading to delayed scaling actions.
Failing to set appropriate thresholds for scaling, which may lead to either under-scaling (reducing capacity when needed) or over-scaling (increasing capacity unnecessarily).
Not adequately testing the system under various load conditions.

#### Follow-Ups
How would you handle scenarios where the OPM metric spikes due to transient events?
What measures would you take to ensure data consistency and reliability in a distributed environment?

**Comparison:**

```
Metric	Description
OPM	Orders per minute monitored metric
```

**Code:**

```

from prometheus_client import start_http_server, Counter

order_counter = Counter('orders_per_minute', 'Number of orders per minute')

def process_order(order): order_counter.inc()

CODE
Copy
```bash
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic orders-topic
```

**Data:**

```json
{
  "retries": 3,
  "batch.size": 16384
}
```

**Flow:**

1. CloudWatch Exporter
2. Kafka Topic
3. Stream Processing (KafkaConsumers)
4. Data Published to MetricsService
5. AWS Auto Scaling Policy

