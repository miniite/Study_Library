---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd migrate ingestion state so switching a connector from full-refresh to incremental doesn't re-sync everything.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Incremental Sync and Connector Migration](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Incremental_Sync_and_Connector_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this migration scenario, we need to understand that moving a data connector from a full-refresh mode to an incremental one can indeed cause issues if not managed properly, as it might lead to unnecessary re-synchronization of large datasets. The core idea is to migrate the ingestion state in such a way that only necessary changes are processed during the switch-over period.

This question primarily tests your understanding of how data ingestion states work, especially when transitioning between full-refresh and incremental modes. We need to ensure minimal disruption by preserving critical metadata related to where we left off with the previous sync. Here’s a step-by-step solution:

Current State Understanding: Identify the current state of the connector, whether it is in full-refresh or incremental mode.
Metadata Migration: Extract and preserve the necessary metadata that will be used for incremental processing.
Stateful Ingestion: Configure Airbyte/Fivetran to handle the transition smoothly without re-processing everything.

