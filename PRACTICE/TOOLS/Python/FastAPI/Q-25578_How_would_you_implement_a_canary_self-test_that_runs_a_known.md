---
type: "practice"
difficulty: "Deep"
---

# How would you implement a canary self-test that runs a known input through the newly loaded model inside FastAPI at startup and refuses to serve if the output deviates from a golden value?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Monitoring Model Quality in APIs](<../../../../TOOLS/Python/FastAPI/Monitoring_Model_Quality_in_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that your FastAPI-based machine learning (ML) application starts in a healthy state, we can implement a canary self-test that runs known input data through the newly loaded model at startup and refuses to serve if the output deviates from expected values. This approach helps catch issues early before users encounter problems.

The core idea is to:

Load the new model during startup.
Run predefined test inputs against this new model.
Compare the outputs with known golden values.
Refuse to start serving requests if there are discrepancies, ensuring a stable and reliable application.

