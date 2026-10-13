---
type: "practice"
difficulty: "Expert"
---

# You're setting the org's messaging strategy for the next three years across 40 teams. How do you decide between one central paved-road Kafka platform, a mesh of per-domain brokers, and letting teams self-select? What governs the decision and how do you sequence the rollout?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Tiered Storage for Logs](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Tiered_Storage_for_Logs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding on a messaging strategy across 40 teams over the next three years, we need to balance centralization with decentralization, ensuring that our architecture is scalable, maintainable, and aligns with organizational goals. The primary decision points are whether to use one central Kafka platform, a mesh of per-domain brokers, or allow self-selection by teams. Each option has its trade-offs in terms of complexity, flexibility, and risk.

Firstly, let's lay out the key criteria:

Scalability: How will the system handle growth?
Reliability and Resilience: Can we ensure message delivery even under failure scenarios?
Maintainability: Is it easy to manage and evolve the infrastructure?
Cost: What are the financial implications of each option?
Team Autonomy vs. Centralization: How much control do teams need, and how centralized should our architecture be?

The central Kafka platform offers a strong candidate for a unified messaging strategy due to its scalability and reliability features. However, it requires significant upfront investment in setting up and maintaining the infrastructure.

A mesh of per-domain brokers might offer more flexibility but could lead to fragmentation and increased complexity as each team manages its own setup.

Allowing self-selection by teams provides maximum flexibility but risks creating a fragmented architecture that is harder to manage centrally.

To sequence the rollout, we should start with foundational components, establish best practices, and gradually introduce central controls before moving towards full-scale implementation or decommissioning existing brokers. Let’s dive into each option in detail.

**Comparison:**

| Decision Criteria        | Score for Central Kafka Platform | Score for Mesh of Per-Domain Brokers | Score for Self-Selection by Teams |
|--------------------------|---------------------------------|--------------------------------------|-----------------------------------|
| Scalability              | 8                               | 7                                    | 6                                 |
| Reliability and Resilience | 9                              | 8                                    | 7                                 |
| Maintainability          | 7                               | 5                                    | 3                                 |
| Cost                     | 6                               | 4                                    | 2                                 |

**Example input / output:**

| Option              | Scalability | Reliability & Resilience | Maintainability | Cost |
|---------------------|-------------|-------------------------|-----------------|------|
| Central Kafka       | 8           | 9                       | 7               | 6    |
| Mesh of Brokers     | 7           | 8                       | 5               | 4    |
| Self-Selection      | 6           | 7                       | 3               | 2    |

**Data:**

```json
{
  "criteria": [
    { "name": "Scalability", "score": 8 },
    { "name": "Reliability and Resilience", "score": 9 },
    { "name": "Maintainability", "score": 7 },
    { "name": "Cost", "score": 6 }
  ],
  "options": [
    {
      "option": "Central Kafka Platform",
      "criteriaScores": [8, 9, 7, 6]
    },
    {
      "option": "Mesh of Per-Domain Brokers",
      "criteriaScores": [7, 8, 5, 4]
    },
    {
      "option": "Self-Selection by Teams",
      "criteriaScores": [6, 7, 3, 2]
    }
  ]
}
```

**Flow:**

1. Central Kafka Platform
2. Does the team require highscalability?
3. Yes
4. Implement Centralized Control
5. No
6. Maintain Local Brokers
7. Mesh of Per-Domain Brokers
8. Does each domain needindependent control?
9. Yes
10. Deploy Domain-SpecificBrokers
11. No
12. Use Centralized Platform
13. Self-Selection by Teams
14. Would teams benefit fromflexibility?
15. Yes
16. Allow Teams to ChoosePlatforms
17. No
18. Adopt Centralized Strategy

