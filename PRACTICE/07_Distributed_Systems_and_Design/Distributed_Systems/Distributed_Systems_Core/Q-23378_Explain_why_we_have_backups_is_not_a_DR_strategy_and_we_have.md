---
type: "practice"
difficulty: "Expert"
---

# Explain why 'we have backups' is not a DR strategy and 'we have a hot standby' is not automatically resilient. Walk me through the failure modes each hides (untested restores, correlated corruption replicated to the standby, failover that never fires) and how you'd validate real recoverability.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Slow Dependencies and Load Shedding](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/Slow_Dependencies_and_Load_Shedding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, having backups is often considered a basic form of data protection, but it does not constitute an effective disaster recovery (DR) strategy on its own. Similarly, maintaining a hot standby can provide some level of availability, but it is insufficient without robust validation and failover mechanisms. Let's dive into the failure modes each strategy hides and discuss how to validate real recoverability.

#### Triage
What Are We Probing?
Backups: The question aims to highlight why simply having backups does not ensure reliable disaster recovery.
Hot Standby: It probes understanding of the limitations of a hot standby setup without proper validation mechanisms.
Failure Modes: We need to explore potential issues like untested restores and correlated data corruption.
Resilience Validation: The importance of thoroughly testing failover mechanisms.
TL;DR

We'll explain why having backups is insufficient for DR, how a hot standby can lead to false security, identify failure modes such as untested restores and correlated data corruption, and discuss the necessity of validating recoverability through rigorous testing.

#### First-Principles Solution
Backups: The Problem with Unverified Restores
Definition: A backup is a copy of data that can be used to restore the original state in case of failure.
Failure Modes:
Unverified Restores: Backup systems often lack mechanisms to verify the integrity and correctness of restored data before relying on it as a failover target.
Correlated Corruption: In cases where backups are not properly isolated, corrupt or erroneous data might be replicated across multiple backup copies, leading to inconsistent states during recovery.
Hot Standby: False Security
Definition: A hot standby is a system in active use that can take over as the primary system in case of failure.
Failure Modes:
Failover Never Fires: If failover mechanisms are not tested regularly, they might not function when needed due to configuration errors or miscommunication between systems.
Correlated Corruption Replication: Corrupted data from a failed primary can be replicated to the hot standby, making it equally unreliable.
Validating Real Recoverability
Regular Testing: Frequent and thorough testing of failover mechanisms is essential to ensure that backups and hot stands are truly viable in critical scenarios.
Isolated Environments: Backups should be stored in isolated environments with no chance of accidental corruption or data mixing.
Failover Mechanisms: Failover protocols must be rigorously tested under simulated failure conditions, including network partitions and hardware failures.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Closing Summary
Both backups and hot stands are valuable components in a comprehensive disaster recovery strategy, but they alone do not guarantee resilience. Unverified restores, correlated data corruption, and untested failover mechanisms pose significant risks that must be addressed through thorough validation.

#### Common Mistakes/Follow-Ups

Assuming that simply having backups or a hot standby is enough.
Overlooking the importance of regular testing and isolation in backup processes.
Failing to implement robust failover protocols and test them under real-world conditions.

Would you like to explore any specific technologies, tools, or best practices related to these concepts?

**Comparison:**

```
Test Result	Backup System	Hot Standby System
Pass/Fail
2023-10-01	Pass	Failed
2023-10-08	Fail	Passed
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Primary System] --> B[Backup System];
    C[Hot Standby System] --> D[Failover Mechanism];
    E{Test Failover} --> F[Pass/Fail];
```

**Example input / output:**

```text
Input: Test failover mechanism for primary system.
Output: Failover successfully completed without data corruption.

Request: Verify hot standby system during planned downtime.
Response: Standby system failed to take over due to configuration issues.
```

**Data:**

```yaml
# Example Schema for Backup Validation
backup_validation:
  test_results: [
    {timestamp: "2023-10-01", result: "pass"},
    {timestamp: "2023-10-08", result: "fail"}
  ]
```

**Flow:**

1. Initiate Backup
2. Store Isolated Backups
3. Frequent Testing
4. Identify and Correct Issues
5. Primary Failure
6. Failover to Hot Standby
7. Failover Test Success
8. Regular Validation Passes
9. Failover Test Fail
10. Correct Configuration Errors

