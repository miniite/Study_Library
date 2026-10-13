---
type: "practice"
difficulty: "Deep"
---

# You need an idempotent, exactly-once retraining trigger for the AIOps models: a drift alarm that fires repeatedly during an incident must launch exactly one expensive retrain, not a herd. Explain the dedup key and lease design so a restarting orchestrator can't spawn duplicate training jobs.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Idempotent Ingestion](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To achieve an idempotent, exactly-once retraining trigger in AIOps models, we need to ensure that even if a drift alarm fires repeatedly during an incident, only one expensive retrain is triggered. This requires a robust deduplication mechanism and a lease-based strategy for orchestrating the training jobs.

The core idea here is to use a "dedup key" that uniquely identifies each potential retraining request. Additionally, we will implement a lease mechanism to ensure that once a job starts, no other instance can start until the current one completes or times out. This design ensures idempotency and prevents duplicate training jobs from being spawned.

