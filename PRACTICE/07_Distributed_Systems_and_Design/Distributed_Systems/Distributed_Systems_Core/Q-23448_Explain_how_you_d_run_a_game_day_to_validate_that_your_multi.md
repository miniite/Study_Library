---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd run a game day to validate that your multi-AZ RDS failover actually meets its RTO. What do you inject what do you measure and what commonly-untested assumption tends to break?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cross-AZ Traffic Costs](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Cross-AZ_Traffic_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure our multi-AZ RDS failover meets the Recovery Time Objective (RTO) during a game day, we'll simulate a failover in a controlled environment. This involves injecting a deliberate failure into one of the Availability Zones (AZs), measuring key metrics, and validating that the system recovers within acceptable timeframes.

#### Triage
What's Being Probed

The goal is to test the RDS multi-AZ configuration for failover robustness. We're primarily interested in verifying:

Failover Time: Measuring how quickly the secondary AZ takes over.
Data Consistency: Ensuring data consistency across both AZs.
Service Availability: Checking if critical services are unaffected.

#### Assumptions
The RDS instances are properly configured for multi-AZ failover.
Network latency and bandwidth between AZs are within acceptable limits.
No external dependencies (e.g., VPC, other AWS services) introduce additional delays.
Step-by-Step Solution

Environment Setup:

We have a multi-AZ RDS deployment with two AZs: Primary (AZ1) and Secondary (AZ2).
The RDS instance is configured for automatic failover.

Injecting the Failure:

Option 1: Directly shut down the primary RDS instance in AZ1.
BASH
# SSH into an EC2 instance in AZ1
ssh ec2-user@ec2-<public-ip>.compute.amazonaws.com

# Stop the RDS instance
aws rds stop-db-instance --db-instance-identifier <instance-id>
Option 2: Simulate a network outage between AZ1 and AZ2.
BASH
# SSH into an EC2 instance in AZ1
ssh ec2-user@ec2-<public-ip>.compute.amazonaws.com

# Use `ping` or `traceroute` to simulate high latency
ping -c 30 <AZ2-rds-public-endpoint>

Monitoring and Measurement:

RDS Status: Monitor the RDS instance status using AWS CloudWatch.
BASH
# Check RDS instance status via CLI
aws rds describe-db-instances --db-instance-identifier <instance-id>
Failover Notifications: Ensure notifications are set up for RDS failover events.
BASH
# Set up SNS topic for RDS failover notifications
aws sns create-topic --name rd-failover-notifications

# Subscribe email to the SNS topic
aws sns subscribe --topic-arn arn:aws:sns:<region>:<account-id>:rd-failover-notifications --protocol email --notification-endpoint <email>

Data Consistency Verification:

After failover, verify data consistency using tools like mysql commands or AWS DMS.
BASH
# Connect to the new primary instance and run a query
mysql -h <AZ2-rds-public-endpoint> -p<password>
SHOW TABLE STATUS;

Service Availability Testing:

Simulate client requests to ensure no downtime or degraded performance.
BASH
# Use `curl` or `httpie` to send HTTP requests to the application
curl http://<application-url>/

Post-Game Day Analysis:

Document and review findings.
MARKDOWN
| Metric         | Expected Result | Observed Result |
|----------------|-----------------|----------------|
| Failover Time  | ≤ 1 minute      | [actual time]   |
| Data Consistency | ✓             | [pass/fail]    |
| Service Availability | ✓            | [pass/fail]    |

#### Common Mistakes
and Follow-Ups
Failure to Test Network Latency: Ensuring network conditions don't cause delays in failover.
Insufficient Monitoring: Missing key metrics that could indicate issues early.
Overlooking External Dependencies: Failing to account for other services that may be impacted.

#### Closing Summary

By carefully simulating a failover and measuring critical performance indicators, we can ensure our RDS multi-AZ setup meets its RTO. Regular game days are crucial to maintaining system reliability and uncovering potential hidden issues.

**Comparison:**

```
Metric	Expected Result	Observed Result
Failover Time	≤ 1 minute	[actual time]
Data Consistency	✓	[pass/fail]
Service Availability	✓	[pass/fail]
```

**Code:**

```bash
aws rds stop-db-instance --db-instance-identifier <instance-id>
ping -c 30 <AZ2-rds-public-endpoint>
```

