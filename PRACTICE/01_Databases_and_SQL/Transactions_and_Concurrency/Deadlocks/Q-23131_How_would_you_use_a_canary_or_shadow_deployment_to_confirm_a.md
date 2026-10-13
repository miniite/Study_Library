---
type: "practice"
difficulty: "Expert"
---

# How would you use a canary or shadow deployment to confirm a deadlock fix under real traffic before full rollout?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Deadlock Retry Strategy](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Deadlock_Retry_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we'll leverage a canary or shadow deployment approach to confirm that our deadlock resolution is effective without disrupting live traffic. This method allows us to gradually introduce the updated system while monitoring its behavior in parallel with the existing one. Here's how we can implement it:

First, let’s understand what deadlock detection and resolution are. Deadlocks occur when two or more processes (or transactions) are waiting for each other to release resources they need, leading to a situation where none of them can proceed. Detecting deadlocks typically involves monitoring transaction execution logs and identifying cycles in the dependency graph of locks.

For this specific scenario, we will use a canary deployment strategy to test our deadlock fix under real traffic. This approach helps us ensure that the new system behaves as expected before fully rolling it out.

The question is essentially asking how to safely and effectively deploy an updated database schema or application logic (which may include improved deadlock detection and resolution) in a production environment, without causing disruptions. The key aspect here is confirming that the fix works under actual load conditions, which can be challenging due to the dynamic nature of real-world traffic.

#### Assumptions

We have a well-defined canary deployment setup.
Our application or database supports transactional operations and logging.
We have monitoring tools in place to track deadlock events.

Step 1: Set Up Canary Environment

Deploy the updated system code with the deadlock fix into a separate, isolated environment (canary). This can be done using containerization technologies like Docker or Kubernetes.
Ensure this environment mimics production as closely as possible in terms of load and configurations.

Step 2: Gradually Introduce Traffic

Start by directing a small percentage of live traffic to the canary environment. This can range from 5% to 10%, depending on the system’s size and complexity.
Monitor performance metrics, transaction success rates, and deadlock occurrences in both the live and canary environments.

Step 3: Verify Deadlock Resolution

Use logging mechanisms to capture detailed information about transactions and their interactions. Specifically, log lock acquisitions and releases, as well as any detected deadlocks.
Implement a monitoring tool that can analyze these logs in real-time or near-real-time to detect deadlock patterns and ensure the fix is effective.

Step 4: Expand Traffic Gradually

Continue incrementally increasing the traffic directed to the canary environment. This step-by-step approach allows us to observe how changes affect system behavior over time.
If any issues arise, roll back to the previous state until you are confident in the stability of the new implementation.

Step 5: Full Rollout if Confirmed

Once the canary deployment passes all verification steps without deadlocks or other critical issues, proceed with a full rollout by redirecting all traffic to the updated system.
Monitor post-rollout for any anomalies and maintain the ability to quickly revert if needed.

By using a canary deployment strategy, we can effectively test our deadlock resolution mechanism under real-world conditions without impacting the entire user base. This approach ensures that the fix is robust and reliable before being fully implemented. Common mistakes include insufficient monitoring and not having proper rollback mechanisms in place if issues do arise during the canary phase.

#### Common mistakes
include:

Not setting up sufficient monitoring to catch deadlocks or other issues.
Deploying too much traffic too quickly, leading to potential instability.
Failing to have a clear rollback plan in case of unexpected problems.

This solution covers all aspects from setup, gradual rollout, verification, and post-verification steps while addressing common pitfalls.

