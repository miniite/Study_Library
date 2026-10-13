---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between structured logging and plain text logging, and why is JSON logging easier to parse in a Python AIOps pipeline?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Log Filtering Scripts](<../../../../TOOLS/Python/Python_Language/Log_Filtering_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Structured logging involves capturing log data in a well-defined format that can be parsed and analyzed easily. In contrast, plain text logging records logs as free-form strings, making parsing and analysis more challenging. JSON (JavaScript Object Notation) is often used for structured logging because it is human-readable, machine-readable, and easy to parse with Python.

To understand why JSON logging is easier to parse in a Python AIOps pipeline, let's break down the concepts and then walk through an example.

**Code:**

```python
# Example of plain text logging
plain_text_log = "2023-10-01 14:58:17 INFO User 'john' logged in from 192.168.1.1"

# Example of JSON logging
json_log = '{"timestamp": "2023-10-01T14:58:17", "level": "INFO", "user": "john", "ip": "192.168.1.1"}'
```

**Example input / output:**

```text
# Input plain text log (raw data)
"2023-10-01 14:58:17 INFO User 'john' logged in from 192.168.1.1"

# Input JSON log (parsed structure)
{"timestamp": "2023-10-01T14:58:17", "level": "INFO", "user": "john", "ip": "192.168.1.1"}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Plain Text Log
6. Parse Log
7. Parsed Data
8. Manual Parsing or Error
9. JSON Log
10. Parse Log
11. Parsed Structure
12. Auto-Parsing FrameworkSupports

