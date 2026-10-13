---
type: "practice"
difficulty: "Basic"
---

# What is a build step for a Python script that trains a small anomaly-detection model, and what would that step produce?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Performance Regression Gates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Performance_Regression_Gates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting up Continuous Integration/Continuous Deployment (CI/CD) for training a small anomaly-detection model with a Python script, the build step is responsible for executing the core logic of the model training process. This typically involves several sub-steps such as loading data, preprocessing, model training, and validation. The output produced by this build step would include both the trained model artifact and any associated metrics or reports that help evaluate its performance.

