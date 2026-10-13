---
type: "practice"
difficulty: "Intermediate"
---

# How would you clean up a cloud account that has accumulated many unused volumes IPs and load balancers?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Instance Roles and Access](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Instance_Roles_and_Access.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To clean up a cloud account with accumulated unused resources like EBS volumes, IP addresses, and Elastic Load Balancers (ELBs), we'll follow these steps from first principles:

Identify and filter out unused resources.
Automate the cleanup process using scripts and tools.
Ensure compliance and minimal disruption.

The goal is to maintain a clean and optimized environment while ensuring no critical services are affected. We will assume AWS as our cloud provider, but the approach can be adapted for other providers with minor changes.

**Comparison:**

```
Resource Type	Example Output
Unused Volumes	[Volume ID 123, Volume ID 456]
Unused IPs	[Public IP 1.2.3.4, Public IP 5.6.7.8]
Unused ELBs	[ELB ARN arn:aws:elasticloadbalancing:region:account-id:load-balancer/app/my-elb/abcd-efgh-ijkl-mnop]
```

**Code:**

```python
# Example Python script for identifying unused resources in AWS

import boto3

def get_unused_volumes():
    ec2 = boto3.client('ec2')
    volumes = ec2.describe_volumes()
    used_volumes_ids = set([vol['Attachments'][0]['InstanceId'] for vol in volumes.get('Volumes', []) if 'Attachments' in vol])
    unused_volumes = [vol for vol in volumes.get('Volumes', []) if vol['VolumeId'] not in used_volumes_ids]
    return unused_volumes

def get_unused_ips():
    ec2 = boto3.client('ec2')
    addresses = ec2.describe_addresses()
    used_ip_addresses = set([addr['InstanceId'] for addr in addresses.get('Addresses', []) if 'InstanceId' in addr])
    unused_ips = [addr for addr in addresses.get('Addresses', []) if addr['PublicIp'] not in used_ip_addresses]
    return unused_ips

def get_unused_elbs():
    elb = boto3.client('elbv2')
    targets = elb.describe_target_health()
    healthy_targets = set([target['TargetId'] for target in targets.get('TargetHealthDescriptions', []) if target['TargetHealthDescription']['State'] == 'Healthy'])
    unused_elbs = [elb for elb in targets.get('TargetHealthDescriptions', []) if elb['Target']['Id'] not in healthy_targets]
    return unused_elbs
```

**Example input / output:**

```text
unused_volumes = get_unused_volumes()
print("Unused Volumes:", unused_volumes)

unused_ips = get_unused_ips()
print("Unused IPs:", unused_ips)

unused_elbs = get_unused_elbs()
print("Unused ELBs:", unused_elbs)
```

**Flow:**

1. Identify unused resources
2. Get unused volumes
3. Get unused IPs
4. Get unused ELBs
5. Audit and review forcompliance
6. Audit and review forcompliance
7. Audit and review forcompliance
8. Review with stakeholders
9. Review with stakeholders
10. Review with stakeholders
11. Mark for deletion/retention
12. Mark for deletion/retention
13. Mark for deletion/retention
14. Execute cleanup using scripts
15. Execute cleanup using scripts
16. Execute cleanup using scripts

