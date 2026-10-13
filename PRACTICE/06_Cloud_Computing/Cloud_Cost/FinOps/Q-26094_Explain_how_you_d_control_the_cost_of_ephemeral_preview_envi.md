---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd control the cost of ephemeral preview environments spun up per pull request. What defaults, TTLs, and cleanup jobs keep them from becoming permanent spend?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Scaling Cost Governance](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Scaling_Cost_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage cloud costs effectively while supporting the use of ephemeral preview environments in a DevOps context, we need to establish a robust system that enforces cost governance through strategic defaults, time-to-live (TTL) policies, and automated cleanup jobs. This approach ensures that these environments are created for short periods only, preventing them from becoming permanent sources of spend.

The key idea here is to leverage the principles of FinOps—financial operations—to align cloud spending with business goals. By setting up clear rules and automations, we can maintain a balance between developer convenience and financial responsibility. Let’s break this down step by step:

Define the Purpose and Scope: Ephemeral preview environments are typically used for quick code reviews, feature testing, or integration checks. We need to ensure that these environments do not go beyond their intended scope.

Establish Defaults and Policies:

Environment Creation Rules: Define default settings for environment creation, such as the type of instances (e.g., t2.micro), storage limits, network configurations, etc.
Time-to-Live (TTL) Policies: Set a TTL that defines how long these environments should exist before being automatically terminated. For instance, if pull requests are expected to be reviewed within 48 hours, set the TTL to this duration.

Automated Cleanup Jobs:

Develop scripts or use cloud provider services (like AWS Lambda or Cloud Functions) to monitor and terminate environments that exceed their TTL.
Implement a daily or hourly cron job that scans for orphaned resources and terminates them if they are past their TTL.
Use tagging and labeling to identify and manage these environments more effectively.

Monitoring and Alerts:

Set up monitoring tools (like Prometheus, Grafana) to track resource usage and costs associated with ephemeral environments.
Configure alerts that notify stakeholders when a significant amount of cost is being incurred or if there are potential issues with environment management.

Cost-Optimization Strategies:

Utilize reserved instances or spot instances where appropriate to reduce costs without compromising performance.
Implement auto-scaling policies to dynamically adjust resources based on usage patterns.

Documentation and Training:

Document the process and provide training for teams to ensure they understand the cost implications of their actions.
Ensure that developers are aware of best practices, such as only creating environments when necessary and regularly cleaning up unused ones.

By implementing these measures, we can effectively control costs while maintaining the agility needed in a dynamic development environment. Common mistakes include over-allocating resources, neglecting TTL policies, or failing to properly document and train teams on cost management practices.

**Comparison:**

```
InstanceId	State	LaunchTime	Tags	Action
i-0abc123	running	2023-10-01T09:00:00Z	EnvironmentType=preview	Terminate
i-0ijkl890	stopped	2023-10-01T07:30:00Z	-	No Action
```

**Code:**

```python
# Example of a cleanup script using AWS Boto3 for terminating old EC2 instances

import boto3
from datetime import datetime, timedelta

def terminate_old_instances(region='us-west-2', ttl_hours=48):
    ec2 = boto3.resource('ec2', region_name=region)

    now = datetime.now()
    cutoff_time = now - timedelta(hours=ttl_hours)

    for instance in ec2.instances.all():
        if 'EnvironmentType' in instance.tags and \
           instance.state['Name'] == 'running' and \
           instance.launch_time < cutoff_time:
            print(f'Terminating {instance.id}...')
            instance.terminate()
```

**Example input / output:**

```text
Input:
- A list of instances with tags and launch times.

Output:
- Instances marked for termination based on TTL rules.
```

**Data:**

```json
{
  "instances": [
    {
      "InstanceId": "i-0abc123def456gh78",
      "State": {"Name": "running"},
      "LaunchTime": "2023-10-01T09:00:00Z",
      "Tags": [{"Key": "EnvironmentType", "Value": "preview"}]
    },
    {
      "InstanceId": "i-0ijkl890lmn456opq",
      "State": {"Name": "stopped"},
      "LaunchTime": "2023-10-01T07:30:00Z"
    }
  ]
}
```

**Flow:**

1. Environment Creation
2. TTL Check
3. No TTL Exceeded
4. Continue Monitoring
5. C
6. Monitor for Costs and Alerts

