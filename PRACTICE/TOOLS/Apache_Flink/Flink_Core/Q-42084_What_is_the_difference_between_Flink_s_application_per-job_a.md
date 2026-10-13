---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between Flink's application, per-job, and session deployment modes, and which fits a multi-team ingestion platform?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Zero-loss Job Upgrades](<../../../../TOOLS/Apache_Flink/Flink_Core/Zero-loss_Job_Upgrades.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: In Apache Flink, applications can be deployed in different modes—application, per-job, or session—to suit various deployment scenarios. For a multi-team ingestion platform, understanding these modes is crucial to ensure flexibility and efficient resource management.

#### Triage
The question asks for the differences between application, per-job, and session deployment modes in Flink and their suitability for a multi-team ingestion platform. We need to explore each mode's characteristics and how they align with the requirements of such a platform.

#### First-Principles Solution

Application Mode: This is the default mode where an entire application runs as one job on the Flink cluster. It’s suitable for applications that require fine-grained control over resource allocation.
Per-Job Mode: Each subtask of the Flink job can be submitted and managed independently. This allows better load balancing and resource utilization, making it ideal for distributed, large-scale processing.
Session Mode: Jobs are split into logical units called sessions, which share resources between them but have their own separate execution contexts. This mode is beneficial when there's a need to isolate jobs from each other while still sharing resources.

For a multi-team ingestion platform:

Each team may require different resource allocations and isolation levels.
Flexibility in job scheduling and management is crucial.
Ensuring that one team’s job failures do not affect others is important.

#### Closing Summary
The choice between application, per-job, or session deployment modes should be based on the specific requirements of the multi-team ingestion platform. Application mode offers tight control over resources but might lack flexibility. Per-job mode provides better resource utilization and isolation for large-scale tasks, making it a good fit for many scenarios in a multi-team environment. Session mode balances resource sharing with job isolation, ensuring that one team’s issues do not impact others.

#### Common Mistakes/Follow-Ups

Mistake: Choosing application mode when per-job or session modes would provide better flexibility and resource management.

#### Follow-Up
Discussing how to manage the lifecycle of jobs in each deployment mode, such as failover strategies and job checkpointing.

