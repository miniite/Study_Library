---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd instrument a Node/Express API so that every DB query's plan-hash and duration are captured, letting you catch plan regressions before users report them.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Index Strategy and Governance at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure we can capture DB query plans and durations in our Node/Express API, we need to instrument our application to log relevant metrics at each database interaction point. This is crucial for identifying plan regressions before they impact user experience. Let's break down the steps from first principles to achieve this.

The key idea is to introduce middleware or custom decorators that wrap every database operation, capturing and logging the query details and execution time. We will leverage Node/Express features like middleware functions and perhaps utilize a performance monitoring tool like New Relic or Jaeger for more sophisticated tracing.

