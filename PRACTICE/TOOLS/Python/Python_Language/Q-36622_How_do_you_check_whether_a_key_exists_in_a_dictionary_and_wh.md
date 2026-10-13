---
type: "practice"
difficulty: "Basic"
---

# How do you check whether a key exists in a dictionary, and why is 'key in d' preferred over 'd.has_key(key)'?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String Immutability and Performance](<../../../../TOOLS/Python/Python_Language/String_Immutability_and_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To check if a key exists in a Python dictionary, there are two common methods: using key in d or d.has_key(key). However, key in d is generally preferred over d.has_key(key) due to its simplicity and compatibility with newer versions of Python.

The core idea here is understanding the syntax and performance implications of these operations. Let's break it down step by step.

