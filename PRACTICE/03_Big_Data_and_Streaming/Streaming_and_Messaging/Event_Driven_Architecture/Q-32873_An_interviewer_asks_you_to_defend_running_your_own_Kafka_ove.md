---
type: "practice"
difficulty: "Deep"
---

# An interviewer asks you to defend running your own Kafka over managed. Six months later the pager burden is crushing your platform team. At what signal do you reverse the decision, and how do you migrate to managed without admitting the original call was wrong in a way that loses the team's trust?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Tiered Storage for Logs](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Tiered_Storage_for_Logs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When making the switch from running our own Kafka cluster to a managed service, we initially believed it would provide scalability and operational ease. However, six months down the line, the platform team is struggling with the pager burden and the associated costs. To defend this decision and ensure a smooth migration without admitting any initial error, we need to establish clear signals for when to reverse course and implement a meticulous plan that honors our team's efforts.

The primary signal to switch back would be an unsustainable increase in operational overhead. This could manifest as:

Frequent, critical system outages.
High pager activity from the platform team.
Elevated costs due to inefficient resource utilization or unexpected charges.

Given these signals, we must approach the migration with a deep understanding of our current state and a well-architected plan. The key steps include assessing the managed service's capabilities, conducting a thorough risk analysis, and ensuring minimal disruption during the transition.

