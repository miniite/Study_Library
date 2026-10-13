---
type: "practice"
difficulty: "Expert"
---

# How would you defend a Python LLM app against prompt injection when retrieved documents or tool outputs contain adversarial instructions, and why is escaping alone insufficient?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Prompt Injection Defense](<../../../../TOOLS/Python/Python_Language/Prompt_Injection_Defense.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To defend a Python LLM app against prompt injection when retrieved documents or tool outputs contain adversarial instructions, we must address the core issue of ensuring that any input derived from external sources is sanitized and validated before it reaches the LLM. The key idea here is to implement robust input validation and sanitization techniques combined with dynamic analysis during runtime.

The question probes several aspects: understanding the nature of prompt injection attacks, recognizing the limitations of escaping characters alone, and designing a comprehensive defense strategy that includes validation, normalization, and anomaly detection mechanisms. We'll start by laying out the fundamental principles before diving into specific solutions.

