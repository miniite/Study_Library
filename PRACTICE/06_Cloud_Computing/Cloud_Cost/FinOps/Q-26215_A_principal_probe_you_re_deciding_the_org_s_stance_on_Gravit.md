---
type: "practice"
difficulty: "Expert"
---

# A principal probe: you're deciding the org's stance on 'Graviton/ARM-first and spot-first as an opt-out default' baked into the paved road. Some workloads legitimately can't comply. Design the default, the exception process, and the tooling that makes the cheap path the path of least resistance without a bureaucratic waiver process.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost-aware Defaults](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost-aware_Defaults.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In this scenario, we need to design a strategy for adopting Graviton/ARM and spot instances as the default compute options in our organization while allowing exceptions where necessary. The goal is to make compliance with cost governance policies straightforward by creating an automated and efficient process that minimizes bureaucratic overhead.

#### Triage
Understanding the Question

The question asks us to design:

Default Compute Strategy: Graviton/ARM and spot instances should be the default.
Exception Process: A mechanism for handling workloads that cannot comply with these defaults.
Tooling: Implementations that ensure compliance by making it easy and efficient, thus reducing bureaucracy.

#### First-Principles Solution
Default Compute Strategy

Key Idea: Automate the provisioning of Graviton/ARM and spot instances as the default choice, leveraging automation tools to minimize manual intervention.

Instance Type Selection: Use AWS Cost Explorer and EC2 instance metadata to automatically select Graviton/ARM and spot instances based on cost-effectiveness.
Resource Scheduling: Implement a service like Amazon EC2 Spot Fleet or Auto Scaling Groups (ASGs) that dynamically adjust the instance types based on cost savings opportunities.
Exception Process

Key Idea: Create a streamlined process for requesting exceptions, ensuring that compliance remains a priority but not overly burdensome.

Request Form: Design an intuitive and user-friendly form for developers to request non-compliant instances, with clear justifications required.
Approval Workflow: Implement an approval workflow using AWS Lambda or similar automation tools to handle exception requests quickly.
Monitoring and Alerts: Set up monitoring and alerts to detect and address any deviations from the default strategy proactively.
Tooling

Key Idea: Develop a comprehensive suite of tools that make compliance effortless, reducing the need for manual intervention and bureaucratic processes.

Automation Framework: Use CloudFormation or Terraform templates to define standard instance types and automate deployment.
Cost Management Dashboard: Create a dashboard using AWS Cost Explorer or similar services that provides real-time cost insights, helping developers make informed decisions.
Compliance Tools: Develop custom tools for monitoring compliance status and providing recommendations for optimizing resource usage.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, we have designed a strategy where Graviton/ARM and spot instances are the default compute options with an efficient exception process and robust tooling to ensure compliance. This approach balances cost optimization with practical considerations, reducing bureaucratic overhead while maintaining control over cloud spend.

#### Common Mistakes

Over-Complexification of Exceptions: Over-complicating the exception request process can lead to delays and inefficiencies.
Ignoring Monitoring and Alerts: Failing to set up adequate monitoring can result in unnoticed deviations from cost governance policies.
Lack of Automation: Relying too heavily on manual processes can introduce errors and increase operational costs.

#### Follow-Ups

Continuous Optimization: Regularly review and update the default instance types based on new pricing models and workload requirements.
Feedback Loop: Implement a feedback loop to gather developer input and refine the tooling and exception process over time.

**Comparison:**

```
Instance Type	Price per Hour
t3.micro	$0.017
t3.nano	$0.0085
Exception Request Form Fields	Description
Workload ID	Unique identifier for the workload.
Justification	Reason why the default instance type cannot be used.
Developer Name	Name of the developer requesting the exception.
Approval Status	Initial status (e.g., pending, approved, denied).
```

**Code:**

```python
# Example Code for Setting Up EC2 Spot Fleet

import boto3

ec2 = boto3.client('ec2')

def launch_spot_fleet():
    response = ec2.request_spot_fleet(
        SpotFleetRequestConfig={
            'IamFleetRole': 'arn:aws:iam::123456789012:role/spotfleet-role',
            'TargetCapacity': 10,
            'LaunchSpecifications': [
                {
                    'ImageId': 'ami-0c55b159cbfafe1f0',
                    'InstanceType': 't3.micro',
                    'WeightedCapacity': 1
                },
                # Additional specifications can be added here
            ],
            'SpotPrice': '0.02'
        }
    )
    print(f"Spot Fleet Request ID: {response['SpotFleetRequestId']}")
```

**Example input / output:**

```text
Input:
- Request for a new EC2 instance with specific criteria (e.g., type, region).

Output:
- Provisioning of an EC2 instance using the Spot Fleet service.
- Response from AWS indicating successful request or any errors.
```

**Data:**

```json
{
  "instanceTypes": ["t3.micro", "t3.nano"],
  "minTargetCapacity": 1,
  "maxTargetCapacity": 10,
  "launchSpecifications": [
    {
      "ImageId": "ami-0c55b159cbfafe1f0",
      "InstanceType": "t3.micro",
      "WeightedCapacity": 1
    },
    {
      "ImageId": "ami-0c55b159cbfafe1f1",
      "InstanceType": "t3.nano",
      "WeightedCapacity": 0.5
    }
  ],
  "spotPrice": "0.02"
}
```

**Flow:**

1. Yes
2. No
3. Approved
4. Denied
5. Initial Request
6. Is Default Instance Type?
7. Provision Default Instance
8. Submit Exception RequestForm
9. Approval Workflow
10. Provision Non-Default Instance
11. Revert to Default Instance orNotify Developer

