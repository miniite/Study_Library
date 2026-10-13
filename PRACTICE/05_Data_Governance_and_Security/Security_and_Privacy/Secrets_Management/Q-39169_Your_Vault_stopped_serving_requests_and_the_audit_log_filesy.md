---
type: "practice"
difficulty: "Expert"
---

# Your Vault stopped serving requests and the audit log filesystem was full. Why did Vault block, and how do you prevent this?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Keeping Secrets out of Env and Logs](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Keeping_Secrets_out_of_Env_and_Logs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a secrets management system like HashiCorp Vault, if it stops serving requests due to an audit log filesystem being full, it indicates that the system is prioritizing data integrity and security over service availability. This behavior ensures that sensitive data isn't lost or compromised during critical operations. Let’s break down why this happens and how we can prevent it.

#### Triage
Understanding the Issue

Vault's primary purpose is to securely manage secrets such as passwords, API keys, etc., in a production environment. The audit log filesystem issue suggests that the system has reached its storage limit for audit logs. Vault uses these logs to track every request, which helps in compliance and troubleshooting. When the filesystem is full, Vault can enter an error state or block all requests.

Key Assumptions:
Audit Logging: Vault is configured to write detailed information about each API request.
Filesystem Fullness: The audit log filesystem has reached its capacity, preventing further writes.

#### First Principles Solution

From first principles, we need to understand the following:

Why Does Vault Block?

When the filesystem fills up, Vault cannot write new logs.
This can lead to a situation where Vault does not have enough resources to process incoming requests due to backpressure from pending log writes.

What Can Go Wrong?

Without writing audit logs, we lose crucial information about request history.
There could be an accumulation of unlogged requests, leading to potential security vulnerabilities or compliance issues.

How Do We Prevent This?

Ensure the filesystem is sized appropriately and monitored for fullness.
Implement a backup strategy for logs.
Configure Vault to handle high volumes without overwhelming the storage system.
Detailed Steps

Filesystem Size and Monitoring:

Set up proper monitoring for the audit log directory size.
Use tools like df on Linux or similar commands in your environment to check disk usage.
Implement alerts when the filesystem approaches its threshold, so you can take proactive measures.

Log Rotation and Archiving:

Configure Vault to automatically rotate logs. This involves setting up log rotation policies that move older logs to a separate storage location.
Use archive commands or tools like logrotate for periodic archiving of old logs.

Filesystem Expansion:

If the filesystem is consistently filling, consider expanding it by increasing disk space or using a different storage solution with more capacity.

Logging Strategy Optimization:

Evaluate what information needs to be logged and what can be reduced.
For example, you might only need to log critical operations instead of every single API request.

Backup and Recovery Plan:

Implement a backup plan for logs in case they are lost due to filesystem issues.
Use cloud storage or another robust data store to back up crucial audit data.

#### Closing Summary

By ensuring proper monitoring, implementing log rotation, optimizing logging strategies, and having a backup plan, you can prevent Vault from blocking requests due to full audit log filesystems. This approach not only maintains operational continuity but also ensures that security and compliance requirements are met.

#### Common Mistakes/Follow-Ups

Mistake: Ignoring filesystem size limits and alerts.

#### Follow-up
Set up automated alerts for storage usage and regularly check disk space.

Mistake: Inadequate log rotation or archiving.

#### Follow-up
Implement a robust log management strategy to handle large volumes of data.

**Comparison:**

```
Monitoring Strategy	Description
Disk Usage Alerts	Set up alerts for critical disk usage thresholds.
Log Rotation	Configure log rotation policies to manage storage efficiently.
Filesystem Expansion	Consider increasing disk space or using a different storage solution.
```

**Code:**

```bash
# Example script for monitoring filesystem fullness using df command
#!/bin/bash
LOG_DIR="/var/vault/log"
THRESHOLD="90"

if [ $(df -h $LOG_DIR | awk 'NR==2{print $5}' | sed 's/%//') -ge $THRESHOLD ]; then
  echo "Filesystem fullness is at ${THRESHOLD}% or more. Alerting..."
  # Send alert using email, Slack, etc.
fi
```

**Example input / output:**

```text
Input:
- Filesystem fullness threshold set to 90%
- Log rotation configured every 7 days

Output:
- Alert sent when filesystem usage reaches or exceeds 90%
- Logs rotated and stored in a separate location
```

**Data:**

```json
{
  "config": {
    "path": "/var/vault/config",
    "rotation_interval": "7d"
  },
  "logs": {
    "level": "info",
    "paths": [
      "/var/vault/log/*.log"
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Monitor Disk Usage
4. Threshold Exceeded?
5. Send Alert
6. Take Action (Expand FS, RotateLogs)
7. Continue Monitoring

