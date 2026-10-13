---
type: "practice"
difficulty: "Deep"
---

# What guardrails distinguish an agentic AIOps remediation planner from a deterministic runbook engine, and when is each the right choice?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Limits of Automation](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Closed_Loop_Automation/Limits_of_Automation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively differentiate between an agentic AIOps remediation planner and a deterministic runbook engine, we need to understand their core principles and where they excel in managing automated responses to incidents or anomalies. An agentic system is designed to autonomously make decisions based on complex, dynamic environments, whereas a deterministic runbook engine follows predefined steps without deviation. The key guardrails that distinguish these systems lie in their decision-making processes, flexibility, adaptability, and the level of manual intervention required.

**Comparison:**

```
Feature	Agentic AIOps Planner	Deterministic Runbook Engine
Decision-Making	Machine learning and real-time context analysis	Fixed rules and predefined steps
Flexibility	High, adapting to new information	Low, rigid and requires updates for changes in environment
Human Involvement	Minimal, with periodic audits	High, manual intervention required for each runbook step
Response Time	Real-time, quick adjustments based on feedback loops	Slower, processing time for predefined steps
```

**Code:**

```python
# Example of an agentic AIOps remediation planner and a deterministic runbook engine

def agentic_planner(state, rules):
    # Agentic system uses machine learning to determine the best course of action.
    decision = ml_model.predict(state)
    return execute_action(decision)

def deterministic_engine(runbook):
    # Deterministic engine follows a fixed sequence of steps
    for step in runbook:
        execute_step(step)
```

**Example input / output:**

```

For an agentic planner, the input state might be:

JSON
Copy
{
  "state": {
    "cpu_usage": 85,
    "memory_usage": 90,
    "error_logs": ["Error code 123", "Error code 456"]
  }
}

The output action could be:

PYTHON
Copy
execute_action('reduce_memory_consumption')

For a deterministic engine, given the runbook:

JSON
Copy
{
  "runbook": [
    {"check_cpu_limit": 90},
    {"reduce_memory_consumption": true}
  ]
}

It would follow these steps in order.
```

**Data:**

```json
{
  "state": {
    "cpu_usage": 85,
    "memory_usage": 90,
    "error_logs": ["Error code 123", "Error code 456"]
  },
  "runbook": [
    {"check_cpu_limit": 90},
    {"reduce_memory_consumption": true}
  ]
}
```

**Flow:**

1. Machine Learning Model
2. Decision Making
3. Execute Action
4. Runbook Rules
5. Step Execution
6. NEXT_STEP

