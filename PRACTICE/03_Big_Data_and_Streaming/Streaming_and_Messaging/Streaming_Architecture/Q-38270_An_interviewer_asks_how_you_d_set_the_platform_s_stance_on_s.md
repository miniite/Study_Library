---
type: "practice"
difficulty: "Deep"
---

# An interviewer asks how you'd set the platform's stance on 'streaming joins as a service': do you let any team join any two topics, or curate blessed joins? Reason about the state-cost and blast-radius consequences of the permissive answer at scale.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cardinality Isolation](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Cardinality_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the interviewer's query on how to set the platform’s stance on 'streaming joins as a service', we need to consider two main approaches: allowing any team to join any two topics (permissive) versus curating and blessing specific joins only. The decision hinges on understanding the implications of each approach in terms of state management, cost, and potential blast radius.

Firstly, let's clarify what streaming joins are: they involve combining data from multiple streams in real-time based on some criteria. In a permissive model, any team can define their own join conditions between topics, which could lead to numerous possible configurations. On the other hand, a curated or blessed approach restricts teams to predefined and validated joins, ensuring consistency and reducing the risk of misuse.

**Code:**

```text
# Example of a permissive join definition
joinDefinition = {
  "stream1": "topicA",
  "stream2": "topicB",
  "keyField": "userId",
  "timeWindow": "5s"
}

# Curated join example
blessedJoin = {
  "id": "userActivityStream",
  "definition": {
    "stream1": "activityTopic",
    "stream2": "loginTopic",
    "keyField": "userId",
    "timeWindow": "30s"
  },
  "validationStatus": "approved"
}
```

**Example input / output:**

```

Permissive Model:

PLAINTEXT
Copy
Input1: {"userId": "u1", "activity": "login"}
Input2: {"userId": "u1", "timeSpent": 50}
Output: {"userId": "u1", "lastLoginTime": "2023-10-01T14:00:00Z", "totalActivity": 50}

Curated Model:

PLAINTEXT
Copy
Input1: {"userId": "u1", "activity": "login"}
Input2: {"userId": "u1", "timeSpent": 50, "lastLoginTime": "2023-10-01T14:00:00Z"}
Output: {"userId": "u1", "totalActivity": 50}
```

**Flow:**

1. Permissive Model
2. Can Create Arbitrary Joins
3. High Flexibility
4. Uncontrolled State Growth
5. Teams Can Innovate Freely
6. Increased Complexity, Harderto Debug
7. Higher Cost of Maintenance
8. Blessed Model
9. Predefined and Validated Joins
10. Consistency and Safety
11. Reduced Blast Radius
12. Teams Have Less Freedom
13. Lower State Complexity
14. Reduced Maintenance Cost

