---
type: "practice"
difficulty: "Deep"
---

# Your Vault cluster's Raft storage is filling disk fast. What generates that data and how do you keep it bounded?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Vault Scalability](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Vault_Scalability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage secrets effectively in a Vault cluster, understanding the sources of excessive data growth is crucial. This primarily involves identifying high-frequency write operations or large secret versions. Here's a step-by-step guide to diagnosing and bounding disk usage.

Triaging the Problem
What the Question Is Asking

The question seeks an explanation for why the Raft storage in Vault is filling up rapidly, along with strategies to keep it within bounds. This involves identifying the root causes of excessive data growth, such as frequent write operations or large secret versions, and implementing measures to mitigate them.

#### Assumptions
and Context
Vault Configuration: We are dealing with a production-grade Vault cluster using Raft for high availability.
Monitoring Tools: The monitoring tools (e.g., Prometheus, Grafana) are already set up to alert on storage levels.
Regular Backups: Automated backups of sensitive data are in place but may not be sufficient to manage ongoing growth.
Step-by-Step Solution

Identify High-Frequency Writes:

Audit Logs Analysis: Analyze Vault's audit logs to identify the frequency and nature of write operations.
BASH
vault audit list --log-level DEBUG
Rate Limiting: Implement rate limiting on sensitive paths if they are causing frequent writes.

Optimize Secret Versions:

Version Management Policies: Review Vault's version management policies to ensure unnecessary versions are not being retained.
HCL
path "secrets/*" {
    capabilities = ["create", "read"]
    secret_max_versions = 5
}
Automated Cleanup: Set up automated cleanup jobs using Vault's sys API or a custom script to remove old versions.

Archiving Old Secrets:

Secret Archival: For long-term storage of historical data, consider archiving secrets off-Vault into a separate storage solution.
HCL
path "archive/*" {
    capabilities = ["read"]
    default_lease_ttl = "720h"
    max_lease_ttl = "1440h"
}
Data Archival Script: Write a script to periodically archive old secrets and delete them from Vault.

Optimize Storage Backends:

Raft Tuning: Adjust Raft storage settings for better performance and space efficiency.
INI
[storage "raft"]
  path = "/var/vault/raft"
  cluster_size = 3
  leader_election {
    enabled = true
    ttl = "1m0s"
  }
Compaction: Enable compaction in the Raft storage backend to reduce the size of stored log entries.

Monitoring and Alerts:

Storage Monitoring: Set up continuous monitoring for storage usage.
YAML
rules:
- alert: VaultStorageHigh
  expr: vault_storage_bytes > 10GB
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Vault storage is over 10GB"
    description: "Monitor and take action to reduce excessive growth."
Alerting Strategy: Configure alerts to notify the team when storage approaches critical levels.

#### Closing Summary
and Common Mistakes

By systematically identifying high-frequency writes, optimizing secret versions, archiving old secrets, tuning storage backends, and setting up robust monitoring, we can effectively manage Vault's Raft storage. Common mistakes include neglecting version management policies, ignoring audit logs, and not having a comprehensive archival strategy.

#### Follow-Ups
Alternative Storage Backends: Explore alternative storage backends like Consul or SQL if the current setup is not sufficient.
Data Encryption: Ensure all data at rest is encrypted to comply with security policies.
Audit Trails: Regularly review audit trails for suspicious activity and fine-tune access controls.

**Comparison:**

| Action           | Description                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------------|
| Audit Logs       | Analyze Vault's audit logs for frequent writes.                                                      |
| Rate Limiting    | Implement rate limiting on sensitive paths to reduce write frequency.                                |
| Secret Archival  | Archive old secrets off-Vault to manage storage growth.                                              |
| Raft Tuning      | Adjust Raft settings for better performance and space efficiency.                                    |
| Monitoring       | Set up continuous monitoring and alerts for storage usage.                                          |

**Code:**

```
Example Vault Policy for Secret Archival

path "archive/*" { capabilities = ["read"] default_lease_ttl = "720h" max_lease_ttl = "1440h" }

CODE
Copy

```ini
# Raft Storage Configuration
[storage "raft"]
  path = "/var/vault/raft"
  cluster_size = 3
  leader_election {
    enabled = true
    ttl = "1m0s"
  }
```

**Example input / output:**

```text
Input:
Audit logs from Vault

Output:
Identify frequent write operations and apply rate limiting.
Optimize secret versions by setting max_lease_ttl.
Archive old secrets to reduce storage usage.
Configure compaction in Raft settings for efficient log management.
Set up monitoring and alerting for proactive management.
```

**Data:**

```
Audit Logs Sample
JSON
Copy
{
  "time": "2023-10-05T14:48:07.969Z",
  "level": "INFO",
  "operation": "write",
  "path": "secrets/mysecret",
  "result": "success"
}
```

**Flow:**

1. Start
2. Identify Frequent Writes
3. Analyze Audit Logs
4. Implement Rate Limiting
5. Optimize Secret Versions
6. Set Up Archival Jobs
7. Tune Raft Settings
8. Set Up Monitoring and Alerts

