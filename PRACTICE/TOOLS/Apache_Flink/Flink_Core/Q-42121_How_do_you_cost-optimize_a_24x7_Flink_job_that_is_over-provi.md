---
type: "practice"
difficulty: "Deep"
---

# How do you cost-optimize a 24x7 Flink job that is over-provisioned for peak but mostly idle overnight?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Checkpoint Overhead](<../../../../TOOLS/Apache_Flink/Flink_Core/Checkpoint_Overhead.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To cost-optimize an Apache Flink job that is over-provisioned for peak load and mostly idle during off-peak hours, we need to focus on reducing resource utilization when there's no significant processing demand. This involves both tuning the job configuration and leveraging cost-effective cloud services.

#### Triage

The question asks about optimizing a 24/7 Flink job that is over-provisioned but idle most of the time during off-peak hours. The core objective is to reduce costs while ensuring the job can handle peak loads when needed. This involves understanding resource allocation, task scheduling, and cost optimization techniques.

#### Solution
Resource Allocation: Since the job is mostly idle, we need to dynamically allocate fewer resources (like CPUs and memory) during off-peak hours.
Task Scheduling: We should design the tasks in a way that can be paused or scaled down when not needed, and resumed when peak load arrives.
Cost Optimization Techniques:
Spot Instances: Use AWS Spot instances or equivalent to take advantage of lower-cost computing resources during idle periods.
Autoscaling: Implement an autoscaler to dynamically adjust the number of tasks based on demand.
State Management: Ensure state is managed efficiently so that resuming from a paused state doesn't require extensive setup time.
Step-by-Step Solution
Determine Peak vs Off-Peak Times:
Analyze historical data to identify peak and off-peak times for the job.
Dynamic Resource Allocation:
Use Flink's dynamic task scheduler (if supported) or implement custom logic to reduce resource consumption during off-peak hours.
Spot Instances Configuration:
Configure the cluster to use Spot instances with a minimum bid price that ensures availability but maximizes cost savings.
Autoscaler Implementation:
Develop an autoscaler script that can be triggered by Flink's metrics (e.g., CPU utilization).
State Management and Resilience:
Ensure state checkpoints are taken regularly to facilitate quick recovery during off-peak periods.

#### Closing Summary

By implementing dynamic resource allocation, leveraging spot instances, and using an autoscaler, we can significantly reduce the cost of running a 24/7 Flink job. This approach ensures that resources are only used when needed, minimizing waste and optimizing costs while maintaining performance under peak loads.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the need for state management during off-peak periods.

#### Follow-Up
Discussing how to handle task checkpoints and state recovery efficiently.

Would you like more detail on any of these steps, such as sample code or specific configuration examples?

**Comparison:**

```
Time Period	CPU Utilization (Average)
Off-Peak	5%
Peak	90%
State Checkpoint Interval	Recovery Time (s)
1 hour	30
```

**Code:**

```python
# Example Python script for autoscaling Flink tasks
import boto3

def check_cpu_utilization():
    # Check CPU utilization metrics from AWS CloudWatch
    cloudwatch = boto3.client('cloudwatch')
    response = cloudwatch.get_metric_statistics(
        Namespace='AWS/EC2',
        MetricName='CPUUtilization',
        Dimensions=[{'Name': 'InstanceId', 'Value': 'your-instance-id'}],
        StartTime=datetime.now() - timedelta(minutes=5),
        EndTime=datetime.now(),
        Period=60,
        Statistics=['Average']
    )
    return response['Datapoints'][0]['Average']

def autoscale_flink_jobs(cpu_utilization):
    if cpu_utilization < 20:  # Example threshold
        # Scale down Flink tasks
        pass
    else:
        # Scale up Flink tasks
        pass

if __name__ == "__main__":
    while True:
        cpu_utilization = check_cpu_utilization()
        autoscale_flink_jobs(cpu_utilization)
        time.sleep(60)  # Check every minute
```

**Example input / output:**

```text
Input:
- CPU Utilization: 12% during off-peak hours

Output:
- Flink tasks scaled down to half the normal number of instances.
- Spot instance bid price set at $0.10, ensuring cost savings without compromising availability.
```

**Data:**

```json
{
  "instanceType": "c5.large",
  "bidPrice": "0.10"
}
```

**Flow:**

1. Yes
2. No
3. Check CPU Utilization
4. CPU < 20?
5. Scale Down Flink Tasks
6. Scale Up Flink Tasks
7. Monitor Metrics and Autoscale

