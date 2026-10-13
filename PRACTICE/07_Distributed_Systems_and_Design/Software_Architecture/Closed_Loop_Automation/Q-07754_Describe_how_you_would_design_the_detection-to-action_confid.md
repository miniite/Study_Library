---
type: "practice"
difficulty: "Expert"
---

# Describe how you would design the detection-to-action confidence gating so low-confidence ML signals never trigger destructive automation.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Limits of Automation](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Closed_Loop_Automation/Limits_of_Automation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In designing a closed-loop automation architecture, one critical aspect is ensuring that only high-confidence decisions are made by automated systems to avoid unintended consequences. For this question, we need to focus on implementing confidence gating mechanisms that ensure low-confidence Machine Learning (ML) signals do not trigger destructive actions. Here’s how I would approach it:

Firstly, the goal is to establish a robust system where ML models generate predictions with associated confidence scores. These predictions are then evaluated and either accepted or rejected based on their confidence levels before any action is taken by the automation system.

The core idea here is to create a layered architecture that integrates multiple filters for decision validation, ensuring high reliability of actions initiated by automated systems.

**Code:**

```python
# Example Python code snippet for confidence gating logic

def evaluate_confidence(confidence_score):
    if confidence_score >= 0.95:
        return True
    else:
        return False
```

**Example input / output:**

```

Example input and output:

Input:

JSON
Copy
{
  "prediction": "high",
  "confidence_score": 0.85,
  "timestamp": "2023-10-01T14:30:00Z"
}

Output:

JSON
Copy
{
  "accepted_for_action": false,
  "reason": "Low confidence score (0.85) does not meet threshold for action"
}
```

**Data:**

```

No specific data schema required, but consider the following JSON structure for ML outputs:

JSON
Copy
{
  "prediction": "high",
  "confidence_score": 0.98,
  "timestamp": "2023-10-01T14:30:00Z"
}
```

**Flow:**

1. No
2. Yes
3. ML Prediction
4. Confidence Score > 0.95?
5. Action Rejected
6. Log and Action
7. Send Alert to Human Operatorfor Review

