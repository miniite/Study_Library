---
type: "practice"
difficulty: "Deep"
---

# How do you decide who leads a database ATAM — an internal architect who knows the query patterns and index history intimately or an external evaluator who does not? Discuss the bias-vs-knowledge tradeoff and how you would staff the review team for maximum signal.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Facilitating an ATAM](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Facilitating_an_ATAM.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding who should lead a Database Architectural Trade-off Analysis (ATAM), we need to weigh the biases and knowledge of both internal and external evaluators. An internal architect has intimate knowledge of query patterns, historical index usage, and system limitations, but this can introduce bias towards existing practices or architectures that might not be optimal for new requirements. On the other hand, an external evaluator brings fresh perspectives and unbiased analysis but lacks deep domain knowledge.

To maximize signal from the ATAM, we should staff the review team carefully:

Internal Architect as Lead: The internal architect leads to ensure a deep understanding of current system constraints and data patterns. Their role is to guide the process, share critical context, and help bridge any gaps in external evaluators' knowledge.
External Evaluators for Fresh Perspective: External experts provide unbiased analysis and challenge assumptions. They can suggest innovative solutions that might not be considered due to internal biases or complacency.

The team composition should balance these perspectives:

Lead Internal Architect (Database Lead): Acts as a subject matter expert, guiding the process and providing context.
External Evaluators: Provide fresh eyes and unbiased analysis. They may come from different industries or have specialized skills in newer technologies not yet adopted by the internal team.

**Code:**

```text
# Example Code for ATAM Team Setup
class ATAMTeam:
    def __init__(self, lead_architect, external_evaluators):
        self.lead_architect = lead_architect
        self.external_evaluators = external_evaluators

    def analyze(self):
        # Lead Architect provides context and guides the analysis
        print(f"Context provided by: {self.lead_architect.name}")

        for evaluator in self.external_evaluators:
            print(f"Insights from: {evaluator.name}")

# Example Usage
class InternalArchitect:
    def __init__(self, name):
        self.name = name

class ExternalEvaluator:
    def __init__(self, name):
        self.name = name

# Create Team Members
internal_architect = InternalArchitect("Alex")
external_evaluators = [ExternalEvaluator("Bob"), ExternalEvaluator("Charlie")]

# Set Up ATAM Team
atam_team = ATAMTeam(internal_architect, external_evaluators)
atam_team.analyze()
```

**Example input / output:**

```text
**Input:**
- Internal Lead Architect: Alex, with deep knowledge of query patterns and index history.
- External Evaluators: Bob and Charlie.

**Output:**
- Context provided by: Alex
- Insights from: Bob
- Insights from: Charlie
```

**Data:**

```json
{
  "team": {
    "internalLead": {
      "name": "Alex",
      "knowledge": ["query patterns", "index history"]
    },
    "externalEvaluators": [
      { "name": "Bob" },
      { "name": "Charlie" }
    ]
  }
}
```

