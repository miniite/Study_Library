---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd instrument a Python training pipeline to emit per-stage timing and resource metrics so you can attribute a slow run to data loading vs compute vs checkpointing.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ OpenTelemetry Instrumentation](<../../../../TOOLS/Python/Python_Language/OpenTelemetry_Instrumentation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To instrument a Python training pipeline to emit per-stage timing and resource metrics, we need to focus on three key areas: identifying stages, measuring performance, and logging or reporting those metrics. By breaking down the pipeline into distinct stages like data loading, compute, and checkpointing, we can attribute slow runs to specific parts of the process.

This solution requires a deep understanding of both the Python ecosystem and best practices in software engineering for observability. Let's explore each stage from first principles.

