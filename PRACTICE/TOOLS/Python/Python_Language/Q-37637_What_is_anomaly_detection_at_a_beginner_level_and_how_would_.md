---
type: "practice"
difficulty: "Basic"
---

# What is anomaly detection at a beginner level, and how would you use Python to flag any metric value that is far above the average of the recent values?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Simple Anomaly Detection](<../../../../TOOLS/Python/Python_Language/Simple_Anomaly_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Anomaly detection involves identifying data points or events that are significantly different from the norm within a dataset. At a beginner level, we can implement this by calculating the mean and standard deviation of recent metric values and flagging any value that is far above the average.

In Python, we would start by collecting recent metric values, then calculate their mean and standard deviation. Any value more than a certain threshold (e.g., 2 or 3 standard deviations) from the mean could be flagged as an anomaly. Let's walk through this step-by-step.

