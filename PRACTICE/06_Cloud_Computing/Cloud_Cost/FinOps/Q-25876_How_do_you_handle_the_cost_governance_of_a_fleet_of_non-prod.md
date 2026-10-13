---
type: "practice"
difficulty: "Deep"
---

# How do you handle the cost governance of a fleet of non-production environments (dev, staging, ephemeral preview envs) that collectively cost as much as prod? Design the policy and automation without hurting developer productivity.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Scaling Cost Governance](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Scaling_Cost_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of managing cloud costs in non-production environments while maintaining high developer productivity, we need to design a comprehensive FinOps strategy that integrates both governance policies and automated cost controls. The key is to strike a balance between stringent cost management and minimizing disruptions to development workflows.

#### Triage

The question asks for a detailed approach to handle the significant cloud costs of non-production environments (dev, staging, ephemeral preview) without compromising developer productivity. We need to understand how these environments are used, their typical resource consumption patterns, and the potential financial impact if left unchecked. The interviewer is probing our ability to design policies that align with business objectives while ensuring smooth development processes.

#### Assumptions
Resource Utilization: Non-production environments tend to have variable usage patterns; they may be idle or active at different times.
Developer Workflow: Developers rely on these environments for testing, integration, and deployment.
Cost Thresholds: There is a defined budget limit that needs to be enforced.

#### First-Principles Solution
1. Policy Design

We start by defining clear cost governance policies tailored to the unique characteristics of non-production environments:

Resource Allocation Policies: Define specific resource limits for each environment type (e.g., CPU, memory).
Auto Scaling Rules: Implement auto-scaling based on usage patterns and historical data.
Scheduling Constraints: Schedule resources during off-peak hours to reduce costs.
2. Automation Implementation

Next, we automate the enforcement of these policies using a combination of cloud-native tools and custom scripts:

Cloud Cost Management Tools: Use services like AWS Budgets, Azure Cost Management, or GCP Billing Alerts.
Custom Scripts & APIs: Develop scripts to dynamically adjust resource allocation based on real-time usage metrics.
3. Monitoring & Alerting

Implement robust monitoring and alerting mechanisms:

Real-Time Metrics: Continuously monitor resource usage in non-production environments.
Alerts & Notifications: Set up alerts for threshold breaches, notifying stakeholders via email or Slack.
4. Developer Communication & Training

Ensure developers are aware of the policies and best practices:

Workshops & Documentation: Conduct training sessions and create detailed documentation.
Feedback Mechanisms: Establish channels for developers to provide feedback on the policy implementation.

#### Closing Summary

By combining well-defined cost governance policies, automation, monitoring, and effective communication with developers, we can manage cloud costs in non-production environments efficiently without hampering productivity. This approach ensures that development workflows remain smooth while keeping expenses under control.

#### Common Mistakes
& Follow-Ups
Ignoring Developer Feedback: Not involving developers in the policy design process can lead to resistance.
Over-Automating: Automating too much can introduce complexity and potential issues if not managed correctly.
Failing to Monitor Performance: Over-enforcing policies without monitoring performance impacts can be counterproductive.

#### Example
Policy
YAML
# Example Policy for Dev Environment
---
resources:
  - type: ec2
    max_instances: 5
    min_instances: 1
    auto_scaling: true

monitoring:
  - metric: cpu_usage
    threshold: 70%
    notify_on: breach
    notification_channels:
      - email: developer@example.com
      - slack_channel: #dev-team

**Comparison:**

```
Policy Type	Description
Resource Allocation	Defines maximum and minimum instances for each environment type.
Auto-scaling Rules	Enforces scaling based on real-time usage metrics.
Scheduling Constraints	Schedules resource allocation during off-peak hours to reduce costs.
```

**Code:**

```python
# Example Python Script for Auto-scaling EC2 Instances Based on CPU Usage

import boto3
from datetime import datetime

ec2 = boto3.client('ec2')

def check_cpu_usage():
    response = ec2.describe_instance_status(
        IncludeAllInstances=True,
    )

    for instance in response['InstanceStatuses']:
        if float(instance['InstanceState']['Name']) == 'running':
            cpu_utilization = get_cpu_utilization(instance['InstanceId'])

            if cpu_utilization > 70:
                scale_out()
            elif cpu_utilization < 30:
                scale_in()

def get_cpu_utilization(instance_id):
    # Mock function to fetch CPU utilization
    return 65

def scale_out():
    ec2.run_instances(
        ImageId='ami-0123456789abcdef0',
        MinCount=1,
        MaxCount=5,
        InstanceType='t2.micro'
    )

def scale_in():
    # Mock function to terminate instances
    pass

check_cpu_usage()
```

**Example input / output:**

```text
# Example Input-Output for Monitoring and Alerting

Input (Real-time Metrics):
CPU Usage of Instance i-0123456789abcdef0 - 70%

Output (Action Taken):
Scale Out Event Triggered
New Instance Created with ID: i-0987654321fedcba0
```

**Data:**

```json
{
  "resources": [
    {
      "type": "ec2",
      "max_instances": 5,
      "min_instances": 1,
      "auto_scaling": true
    }
  ],
  "monitoring": [
    {
      "metric": "cpu_usage",
      "threshold": 70,
      "notify_on": "breach",
      "notification_channels": [
        {
          "type": "email",
          "address": "developer@example.com"
        },
        {
          "type": "slack_channel",
          "channel": "#dev-team"
        }
      ]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Detect CPU Usage
5. CPU > 70%?
6. Scale Out
7. Monitor New Instance
8. CPU < 30%?
9. Scale In
10. Monitor Remaining Instances

