---
type: "practice"
difficulty: "Expert"
---

# How does asyncio.TaskGroup (3.11+) change error handling compared to gather, and how do exception groups let you handle multiple concurrent failures?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Asyncio Concurrency](<../../../../TOOLS/Python/Python_Language/Asyncio_Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Python 3.11+, asyncio.TaskGroup introduces a new way of managing and waiting for asynchronous tasks, significantly changing how we handle errors compared to using gather. Specifically, TaskGroup allows for more robust error handling by leveraging exception groups. This change is crucial because it enables us to efficiently manage multiple concurrent failures in an asynchronous context.

To understand the difference between gather and TaskGroup, let's first look at what these tools are designed for and then dive into their key differences, especially regarding exception handling.

