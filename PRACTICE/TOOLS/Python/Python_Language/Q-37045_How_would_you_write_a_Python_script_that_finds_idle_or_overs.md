---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a Python script that finds idle or oversized EC2 instances by correlating CloudWatch CPU metrics with instance types, to drive a rightsizing recommendation?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ OpenTelemetry Instrumentation](<../../../../TOOLS/Python/Python_Language/OpenTelemetry_Instrumentation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this requirement, we'll need to create a Python script that fetches EC2 instance details and their associated CloudWatch CPU utilization metrics, correlates them with the instance type, and then determines if an instance is idle or oversized based on predefined thresholds. This process involves several steps, including interacting with AWS services, parsing metric data, comparing it against typical usage patterns, and making recommendations.

#### Triaging the Question

The question asks for a Python script that:

Fetches information about EC2 instances from AWS.
Retrieves CloudWatch CPU utilization metrics for these instances.
Compares the CPU usage with the instance's type to determine if it is underutilized or overprovisioned.
Generates recommendations based on this comparison.

#### Assumptions
The script will be run in an environment where Python and AWS CLI (or Boto3) are installed.
AWS credentials are configured via aws configure or IAM roles.
We have the necessary permissions to access EC2 instance details and CloudWatch metrics.
Step-by-Step Solution

Fetch Instance Details:

Use Boto3, the AWS SDK for Python, to get a list of all EC2 instances.

Retrieve CPU Utilization Metrics:

Query CloudWatch for recent CPU utilization data for each instance.
Parse and aggregate this data over a period (e.g., last 7 days).

Compare Metrics with Instance Type:

Define thresholds based on typical usage patterns or historical data.
Determine if the instance is underutilized (low CPU usage) or overprovisioned (high CPU usage compared to type).

Generate Recommendations:

Based on the comparison, suggest appropriate actions such as resizing or terminating instances.
Python Code
PYTHON
import boto3
from datetime import datetime, timedelta

def get_instance_details():
    ec2 = boto3.resource('ec2')
    return [instance.id for instance in ec2.instances.all()]

def fetch_cpu_metrics(instance_ids):
    cloudwatch = boto3.client('cloudwatch')
    start_time = (datetime.now() - timedelta(days=7)).isoformat()
    end_time = datetime.now().isoformat()

    metrics_data = []
    for instance_id in instance_ids:
        response = cloudwatch.get_metric_statistics(
            Namespace='AWS/EC2',
            MetricName='CPUUtilization',
            Dimensions=[{'Name': 'InstanceId', 'Value': instance_id}],
            Statistics=['Average'],
            Period=3600,
            StartTime=start_time,
            EndTime=end_time
        )
        metrics_data.append(response['Datapoints'])
    return metrics_data

def analyze_instances(metrics_data):
    recommendations = []
    for instance_id, data in zip(instance_ids, metrics_data):
        avg_cpu_usage = sum(point['Average'] for point in data) / len(data)

        # Example thresholds
        if avg_cpu_usage < 10:  # Low CPU usage threshold
            recommendations.append((instance_id, "Idle"))
        elif avg_cpu_usage > 95:  # High CPU usage threshold
            recommendations.append((instance_id, "Overprovisioned"))
    return recommendations

def main():
    instance_ids = get_instance_details()
    metrics_data = fetch_cpu_metrics(instance_ids)
    recommendations = analyze_instances(metrics_data)

    for instance_id, status in recommendations:
        print(f"Instance {instance_id} is {status}")

if __name__ == "__main__":
    main()

#### Closing Summary

This script provides a basic framework for identifying idle or oversized EC2 instances by correlating CloudWatch metrics with instance types. It leverages Boto3 to interact with AWS services, fetches relevant data, and generates recommendations based on predefined thresholds.

#### Common Mistakes/Follow-Ups
Incorrect Time Periods: Ensure the time period used for fetching metrics is appropriate.
Threshold Definitions: Fine-tune threshold values based on actual usage patterns.
Security Considerations: Ensure proper IAM roles and permissions are in place to avoid security issues.
Scalability: For large-scale environments, consider optimizing queries or using batch processing.

**Code:**

```python
import boto3
from datetime import datetime, timedelta

def get_instance_details():
    ec2 = boto3.resource('ec2')
    return [instance.id for instance in ec2.instances.all()]

def fetch_cpu_metrics(instance_ids):
    cloudwatch = boto3.client('cloudwatch')
    start_time = (datetime.now() - timedelta(days=7)).isoformat()
    end_time = datetime.now().isoformat()

    metrics_data = []
    for instance_id in instance_ids:
        response = cloudwatch.get_metric_statistics(
            Namespace='AWS/EC2',
            MetricName='CPUUtilization',
            Dimensions=[{'Name': 'InstanceId', 'Value': instance_id}],
            Statistics=['Average'],
            Period=3600,
            StartTime=start_time,
            EndTime=end_time
        )
        metrics_data.append(response['Datapoints'])
    return metrics_data

def analyze_instances(metrics_data):
    recommendations = []
    for instance_id, data in zip(instance_ids, metrics_data):
        avg_cpu_usage = sum(point['Average'] for point in data) / len(data)

        # Example thresholds
        if avg_cpu_usage < 10:  # Low CPU usage threshold
            recommendations.append((instance_id, "Idle"))
        elif avg_cpu_usage > 95:  # High CPU usage threshold
            recommendations.append((instance_id, "Overprovisioned"))
    return recommendations

def main():
    instance_ids = get_instance_details()
    metrics_data = fetch_cpu_metrics(instance_ids)
    recommendations = analyze_instances(metrics_data)

    for instance_id, status in recommendations:
        print(f"Instance {instance_id} is {status}")

if __name__ == "__main__":
    main()
```

