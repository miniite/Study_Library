---
type: "practice"
difficulty: "Expert"
---

# How do you version and roll out a change to shared processing logic when it's live in streaming and also embedded in scheduled batch jobs? Walk me through blue/green for a unified pipeline.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Unified Batch and Streaming Platforms](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a unified pipeline where both batch and stream processing share common logic, versioning and rolling out changes can be challenging but manageable with a robust strategy. The key idea is to maintain two environments or "versions" of your shared processing logic—let's call them the "current" and "new" versions—using a blue/green deployment approach. This ensures that you can safely transition from one version to another without downtime.

First, let’s break down what this question is really asking: How do we ensure seamless updates in both batch and stream processing pipelines when they are intertwined? The interviewer will likely probe into the technical details of how changes are managed, potential risks, and best practices for smooth transitions. We'll assume that the current version is live and needs to be updated without disrupting existing operations.

Step-by-Step Solution from First Principles

Define Blue/Green Environments: Create two environments—Blue (current) and Green (new)—for your shared processing logic. This allows you to update one environment at a time while keeping the other active.

Incremental Rollout:

Batch Jobs: Schedule batch jobs on the new version during non-peak hours, ensuring no overlap with existing scheduled runs.
Streaming Logic: Gradually route traffic from the current stream processing logic to the new version by slowly increasing the proportion of data flowing through it.

Monitoring and Testing:

Implement thorough monitoring and automated tests for both environments to catch any issues early.
Use canary releases to test a small subset of the data on the new version before full deployment.

Rollback Plan: Have a clear rollback strategy in place, including backup procedures and steps to quickly revert if something goes wrong.

Communication and Coordination: Ensure all stakeholders are informed about the changes and involved in coordination meetings.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, using blue/green deployment for batch/stream unification ensures a smooth transition by isolating changes and gradually rolling them out. Common mistakes include not having a robust rollback plan or failing to monitor closely during the transition phase. Be prepared for follow-up questions on specific technologies used, potential performance impacts, and how to handle data consistency across environments.

**Flow:**

1. Initial State
2. Blue Version Active
3. C
4. Monitor Performance
5. Full Green Deployment
6. Rollback if Necessary

