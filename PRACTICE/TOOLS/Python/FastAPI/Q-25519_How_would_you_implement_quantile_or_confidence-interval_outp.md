---
type: "practice"
difficulty: "Deep"
---

# How would you implement quantile or confidence-interval outputs from a FastAPI regression endpoint, and how do you validate the intervals stay calibrated in production?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Monitoring Model Quality in APIs](<../../../../TOOLS/Python/FastAPI/Monitoring_Model_Quality_in_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement quantile or confidence-interval outputs in a FastAPI regression endpoint, we need to ensure that our model can predict these intervals alongside point predictions. This involves training an ensemble of models or using techniques like bootstrapping or Bayesian methods. The key steps are:

Model Training: Train a regression model to predict the mean and optionally other quantiles.
Prediction Endpoint: Create a FastAPI endpoint that returns both point predictions and interval predictions.
Validation in Production: Monitor the intervals for stability and recalibration as needed.

We'll start by outlining the steps and then dive into the implementation details.

