---
type: "practice"
difficulty: "Intermediate"
---

# How would you get basic system metrics like CPU and memory usage in Python, and what does the psutil library provide?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ OpenTelemetry Instrumentation](<../../../../TOOLS/Python/Python_Language/OpenTelemetry_Instrumentation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To gather basic system metrics like CPU and memory usage in Python, we often use the psutil library. The TL;DR is that psutil simplifies the process by providing a high-level interface to access system information, making it easy for DevOps engineers to monitor system health without dealing with low-level APIs.

The question asks two main things: how to get these metrics and what additional benefits psutil provides beyond basic usage. Let's break this down step-by-step using first principles.

