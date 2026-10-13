---
type: "practice"
difficulty: "Basic"
---

# What is the difference between 'print' and using the logging module in an automation script that runs unattended in a pipeline?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Request Logging for Models](<../../../../TOOLS/Python/Python_Language/Request_Logging_for_Models.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an automation script running unattended, such as within a DevOps pipeline, understanding the differences between print statements and using the Python logging module is crucial. Both can be used to output information, but their use cases, flexibility, and features differ significantly.

The key difference lies in how they handle logs, verbosity, and handling errors, making the logging module more suitable for production environments where reliability and traceability are essential.

**Code:**

```python
import logging

# Using print
print("This is a simple message")

# Configuring logging
logging.basicConfig(level=logging.INFO)
logging.info("This is an info message")
```

