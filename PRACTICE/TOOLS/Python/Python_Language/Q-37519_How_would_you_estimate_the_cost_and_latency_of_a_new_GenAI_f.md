---
type: "practice"
difficulty: "Intermediate"
---

# How would you estimate the cost and latency of a new GenAI feature in Python before building it, from expected token counts, model choice, retrieval calls, and concurrency?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Estimating GenAI Cost and Latency](<../../../../TOOLS/Python/Python_Language/Python_Tail_Latency/Estimating_GenAI_Cost_and_Latency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To estimate the cost and latency of a new GenAI feature in Python before building it, we need to consider several key factors such as expected token counts, model choice, retrieval calls, and concurrency. Let's break down each component step by step.

Token Count: The number of tokens (words or sub-words) is crucial because models are often priced per token.
Model Choice: Different models have different costs and performance characteristics; some may be more efficient in terms of latency but costlier in terms of token count.
Retrieval Calls: These involve fetching data from an external knowledge base, which can impact both cost and latency.
Concurrency: The number of concurrent requests can affect the overall load on the system, impacting both cost and performance.

To estimate these factors, we will start by defining a baseline model and then iteratively refine our estimates based on real-world data.

