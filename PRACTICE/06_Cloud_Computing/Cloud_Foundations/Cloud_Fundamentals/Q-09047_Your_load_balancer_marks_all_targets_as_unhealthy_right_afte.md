---
type: "practice"
difficulty: "Intermediate"
---

# Your load balancer marks all targets as unhealthy right after deploy. What are the common reasons?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Health Checks and Load Balancers](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Health_Checks_and_Load_Balancers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a cloud-based load balancer marks all target instances as unhealthy immediately after deployment, it often indicates issues with network connectivity or configuration. This is a critical problem that needs to be addressed quickly to ensure application availability and minimize downtime.

#### Triage
and Assumptions:
Network Connectivity: Check if there are any network issues preventing the load balancer from reaching the target instances.
Load Balancer Configuration: Verify the health check configuration on the load balancer.
Target Instance Status: Ensure that the target instances are up and running and can handle incoming traffic.
Step-by-Step Solution:

Check Network Connectivity:

Verify Security Groups and Firewalls: Ensure there are no network policies or firewalls blocking the load balancer's health check requests.
BASH
# Example: Check AWS security groups rules for inbound access from ELB to target instances
aws ec2 describe-security-groups --group-ids <security-group-id>
Check Routing Tables and Subnets: Ensure that the subnets are correctly routed and accessible.

Review Load Balancer Health Checks:

HTTP/HTTPS Health Check Configuration: Make sure the health check path, interval, timeout, and unhealthy threshold are set correctly.
BASH
# Example: Modify ALB configuration in AWS Console or using API
aws elbv2 modify-target-group-attributes --target-group-arn <target-group-arn> \
    --attribute HealthCheckPath=/health
TCP/SSL Health Check Configuration: Ensure the health check protocol and port are correctly configured.

Validate Target Instance Status:

Instance State: Confirm that all target instances are running.
BASH
# Example: List AWS EC2 instance states
aws ec2 describe-instances --filters Name=instance-state-name,Values=running
Health and Load: Ensure the instances can handle load without crashing or timing out during health checks.

#### Closing Summary

The common reasons for a cloud-based load balancer marking all targets as unhealthy upon deployment include network connectivity issues, misconfigured health check settings on the load balancer, and target instance states not meeting health requirements. By systematically checking these areas, you can identify and resolve the root cause to ensure optimal application availability.

#### Common Mistakes/Follow-Ups
Overlooking Network Policies: Failing to update or verify security group rules and routing tables.
Misconfigured Health Checks: Not properly setting up interval, timeout, and unhealthy threshold parameters.
Ignoring Instance State: Overlooking the possibility that target instances are not running or handling traffic correctly.

**Comparison:**

| Security Group Rule | Protocol | Port Range | CIDR |
|---------------------|----------|------------|------|
| SSH Access          | TCP      | 22         | 0.0.0.0/0     |
| HTTP and HTTPS      | TCP      | 80, 443    | 10.0.0.0/16   |
| Default Egress      | -1       | -1         | 0.0.0.0/0     |

**Code:**

```bash
# Example commands for troubleshooting

aws ec2 describe-security-groups --group-ids sg-12345678 | jq '.SecurityGroups[].IpPermissions'
```

**Example input / output:**

```text
# Example output of security group rules

$ aws ec2 describe-security-groups --group-ids sg-12345678 | jq '.SecurityGroups[].IpPermissions'
[
  {
    "FromPort": 22,
    "IpProtocol": "tcp",
    "ToPort": 22,
    "UserIdGroupPairs": []
  },
  {
    "FromPort": 80,
    "IpProtocol": "tcp",
    "ToPort": 443,
    "UserIdGroupPairs": [
      {
        "GroupId": "sg-12345678",
        "VpcId": "vpc-abcd1234"
      }
    ]
  },
  {
    "FromPort": -1,
    "IpProtocol": "-1",
    "ToPort": -1,
    "UserIdGroupPairs": [
      {
        "GroupId": "sg-12345678",
        "VpcId": "vpc-abcd1234"
      }
    ]
  }
]
```

**Data:**

```json
{
  "security_groups": [
    {
      "id": "sg-12345678",
      "ingress_rules": [
        { "protocol": "tcp", "port_range": [22], "cidr": "0.0.0.0/0" },
        { "protocol": "http", "port_range": [80, 443], "cidr": "10.0.0.0/16" }
      ],
      "egress_rules": [
        { "protocol": "-1", "port_range": [], "cidr": "0.0.0.0/0" }
      ]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Check Security Groups
6. Are there any rules blockingthe load balancer?
7. Update or remove restrictivesecurity group rules
8. Verify subnets and routingtables
9. Validate Target Instances
10. Are all target instancesrunning?
11. Maintain health checkconfiguration
12. Restart or troubleshoot issueswith target instances
13. Moving Forward: Ensure healthchecks are correctlyconfigured.
14. Monitor and adjust asnecessary.

