---
type: "practice"
difficulty: "Expert"
---

# Your Vault Raft storage is growing unbounded due to lease and token accumulation. Diagnose the architectural causes at scale and design guardrails (TTL policy, batch tokens, lease-count quotas) to bound it permanently.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Vault Scalability](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Vault_Scalability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Vault's Raft storage is growing unbounded due to excessive token and lease accumulation. To address this issue at scale, we need to implement several guardrails such as TTL policies, batch tokens, and lease-count quotas. This solution involves understanding the underlying mechanisms of Vault's token management and lease renewal processes.

#### Triage

The question revolves around diagnosing why Vault's Raft storage is growing unbounded and designing solutions to manage this growth effectively. Specifically, we need to:

Identify the root causes leading to excessive token and lease accumulation.
Design guardrails such as TTL policies, batch tokens, and lease-count quotas.

#### First Principles Solution

Understanding Token and Lease Accumulation:

Tokens: Tokens are issued by Vault for API access. Each token has a limited lifespan (TTL) and can be renewed multiple times.
Leases: Leases allow secrets to be stored with a time-to-live (TTL), after which they expire.

Root Causes of Unbounded Growth:

Tokens are not being properly managed or refreshed, leading to their indefinite retention in the Raft storage.
Leases are not being renewed or deleted when they should, causing them to accumulate over time.

Implementing Guardrails:

TTL Policies
Token TTL: Implement a short TTL for tokens (e.g., 15 minutes) and set up automatic token renewal mechanisms. This ensures that old tokens are periodically refreshed.
BASH
vault write sys/mounts/myapp/tokens/rotate ttl=15m
Lease TTL: Set appropriate TTL values for leases to ensure they do not outlive their usefulness.
JSON
{
  "lease_ttl": "30m",
  "renewable": true
}
Batch Tokens and Lease Management
Batch Tokens: Issue batch tokens that can be used in bulk operations. This reduces the number of individual token requests, minimizing the accumulation.
BASH
vault write sys/mounts/myapp/tokens/issue/batch ttl=15m policies=my-policy
Lease Count Quotas: Implement quotas to limit the total number of leases that can exist at any given time. This helps in bounding the storage usage.
JSON
{
  "path": "sys/mounts/myapp/tokens/issue/batch",
  "capabilities": ["create"],
  "max_lease_count": 1000
}
Monitoring and Logging
Monitoring: Regularly monitor the number of tokens and leases to ensure they stay within acceptable limits.
BASH
vault read sys/mounts/myapp/tokens/usage
Logging: Implement logging for token and lease operations to trace their usage and identify any anomalies.

#### Closing Summary
and Common Mistakes/Follow-Ups

To permanently bound Vault's Raft storage, we need a combination of effective policies and monitoring. Common mistakes include:

Setting overly long TTLs, leading to unnecessary retention.
Failing to implement automatic token renewal mechanisms.
Overlooking the need for lease count quotas.

For follow-ups, consider additional measures like:

Automated Cleanup: Implement automated scripts to clean up expired tokens and leases.
Review Usage Patterns: Regularly review how tokens and leases are being used in your application.
Stress Testing: Conduct stress tests to ensure that the new policies do not introduce any performance bottlenecks.

