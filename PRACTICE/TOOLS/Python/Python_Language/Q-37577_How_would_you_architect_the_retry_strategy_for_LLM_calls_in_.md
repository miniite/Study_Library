---
type: "practice"
difficulty: "Deep"
---

# How would you architect the retry strategy for LLM calls in Python so it respects a per-request deadline budget across attempts, backs off with jitter, and never retries a non-idempotent tool call or a content-policy refusal?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Retries with Deadline Budgets](<../../../../TOOLS/Python/Python_Language/Dead_Letter_and_Replay_Systems/Retries_with_Deadline_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect the retry strategy for LLM (Large Language Model) calls in Python that respects per-request deadline budgets, backs off with jitter, and handles non-idempotent operations and policy refusals, we need to ensure a robust system design. At its core, this involves defining clear failure handling mechanisms, incorporating exponential backoff with jitter, and carefully managing the retry logic to avoid non-idempotent actions or content-policy violations.

We are tasked with crafting an approach that ensures each LLM call respects the deadline, reduces retries over time by introducing random delays (jitter), and avoids repeating certain types of operations. Let's break this down step-by-step from first principles.

