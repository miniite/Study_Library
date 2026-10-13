---
type: "practice"
difficulty: "Intermediate"
---

# You deploy across three AZs behind an ALB but a zonal outage still caused a full outage. Walk me through how cross-zone load balancing zonal shift and unhealthy-host ejection interact and where the design failed.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cross-AZ Traffic Costs](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Cross-AZ_Traffic_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, you have deployed your application across three Availability Zones (AZs) behind an Application Load Balancer (ALB). Despite this setup, a zonal outage still led to a full outage. This indicates that there are issues in the cross-zone load balancing and unhealthy-host ejection mechanisms. Let's break down how these components interact and where the design might have failed.

First, let’s clarify what cross-zone load balancing, zonal shift, and unhealthy-host ejection are:

Cross-Zone Load Balancing: ALB can distribute incoming traffic across multiple AZs to ensure high availability.
Zonal Shift: This involves moving workloads between different zones in response to health checks or other metrics.
Unhealthy-Host Ejection: If a host becomes unhealthy, it is removed from the load balancer’s pool of available hosts.

The interaction between these components should ideally prevent a full outage. However, if a zonal outage still causes a complete failure, there might be misconfigurations or design flaws in one or more of these mechanisms.

#### Triage
and Assumptions
Assumption 1: The ALB is properly configured for cross-zone load balancing.
Assumption 2: Health checks are functioning correctly to identify unhealthy hosts.
Assumption 3: Zonal shift policies are in place and triggered as expected.
Step-by-Step Solution
1. Understanding the Current Setup

You have an ALB configured with cross-zone load balancing, which should distribute traffic across three AZs (AZ A, AZ B, and AZ C).

2. Analyzing the Outage Scenario

Let’s assume a zonal outage occurs in AZ A:

Cross-Zone Load Balancing: The ALB is designed to route traffic from AZ A to AZ B and AZ C.
Health Checks: These should be configured to monitor the health of instances in each zone. If an instance becomes unhealthy, it should be ejected from the load balancer’s pool.
3. Identifying Potential Failures

Given that a zonal outage still caused a full outage:

Zonal Health Check Mechanism: Ensure that the health checks are configured correctly and triggered during a zonal outage.
Ejection of Unhealthy Hosts: Verify that unhealthy hosts in AZ A are being ejected properly, allowing traffic to be redirected to healthy instances in other zones.
Zonal Shift Policies: Confirm that if an entire AZ goes down, the system can automatically shift workloads from that zone to the remaining healthy zones.
4. Detailed Analysis

Let’s walk through a hypothetical scenario:

Initial State:

ALB is configured with three target groups (TG A, TG B, and TG C), each corresponding to an AZ.
Each TG has instances in multiple availability zones for redundancy.

Zonal Outage Scenario:

Assume AZ A experiences a complete failure. Health checks fail for all instances in AZ A.

Expected Behavior:

The ALB should detect the failed health checks and eject the unhealthy hosts from TG A.
Traffic intended for AZ A should be redirected to healthy instances in TG B and TG C.

Actual Outcome:

If a full outage occurs, it means that:
Health checks did not trigger or were misconfigured.
Unhealthy hosts in AZ A were not ejected from the load balancer’s pool.
Zonal shift policies failed to redistribute traffic effectively.
5. Common Mistakes and Solutions
Misconfigured Health Checks: Ensure health check configurations are correct, with appropriate intervals and thresholds.
Delayed Ejection of Unhealthy Hosts: Verify that ejection policies are triggered promptly. Delayed ejections can lead to prolonged exposure to unhealthy instances.
Inadequate Zonal Shift Policies: Ensure zonal shift policies are robust enough to handle failures in multiple zones simultaneously.

#### Closing Summary
and Follow-Ups

The design failure likely lies in the health check mechanisms or zonal shift policies. To prevent full outages, ensure:

Correctly configured health checks.
Timely ejection of unhealthy hosts.
Robust zonal shift policies that can handle simultaneous failures across multiple zones.

#### Common mistakes
include overlooked health check configurations and inadequate response to unhealthy instances during a zonal outage. Follow-ups could involve reviewing the specific configuration details or conducting a detailed failover test to identify and fix any gaps in the design.

**Comparison:**

| AZ  | Target Group | Health Check Status |
|-----|--------------|---------------------|
| A   | TG A         | Unhealthy           |
| B   | TG B         | Healthy             |
| C   | TG C         | Healthy             |

**Code:**

```python
# Example of ALB Health Check Configuration (Python pseudo-code)
def configure_health_check(alb_client, tg_arn):
    response = alb_client.modify_target_group_attributes(
        TargetGroupArn=tg_arn,
        HealthCheckProtocol='HTTP',
        HealthCheckPath='/healthcheck',
        HealthyThresholdCount=3,
        UnhealthyThresholdCount=2
    )
    return response

# Example of Zonal Shift Policy Configuration (Python pseudo-code)
def configure_zonal_shift_policy(alb_client, tg_arn):
    response = alb_client.modify_load_balancer_attributes(
        LoadBalancerArn='lb-arn',
        CrossZoneEnabled=True,
        AccessLogsEnabled=False
    )
    return response
```

**Example input / output:**

```text
Input:
- ALB with three target groups (TG A, TG B, and TG C)
- Health checks configured for each group

Output:
- All instances in AZ A fail health checks
- Unhealthy hosts in AZ A are ejected from the load balancer’s pool
- Traffic is redirected to healthy instances in TG B and TG C

Expected:
- No full outage; traffic is effectively distributed across remaining zones
```

**Data:**

```json
{
  "health_check": {
    "protocol": "HTTP",
    "path": "/healthcheck",
    "timeout": 10,
    "interval": 30,
    "healthy_threshold_count": 3,
    "unhealthy_threshold_count": 2
  },
  "zonal_shift_policy": {
    "cross_zone_enabled": true,
    "access_logs_enabled": false
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. ALB Receives Request
6. Is Host in AZ A Healthy?
7. Forward to Healthy Hosts in TGB and C
8. Are Zonal Shift PoliciesEnabled?
9. Trigger Zonal Shift toDistribute Traffic
10. Fail Over to PreconfiguredBackup Zones
11. Distribute Traffic AcrossRemaining Healthy Zones
12. Initiate Failover toPreconfigured Backup Zones

