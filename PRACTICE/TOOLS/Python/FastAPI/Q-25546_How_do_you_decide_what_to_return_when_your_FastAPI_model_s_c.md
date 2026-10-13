---
type: "practice"
difficulty: "Deep"
---

# How do you decide what to return when your FastAPI model's confidence is low, and how would you build a human-in-the-loop escalation path from the serving layer?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Time-series Prediction APIs](<../../../../TOOLS/Python/FastAPI/Time-series_Prediction_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide what to return when your FastAPI model's confidence is low, we need to consider both the operational requirements of our application and the trade-offs between user experience, system reliability, and operational costs. The key idea here is to implement a human-in-the-loop (HITL) system that escalates decisions made by the automated model to human experts when the confidence level drops below a certain threshold.

FastAPI provides a robust framework for building web APIs, but its primary focus is on simplicity and performance rather than handling complex decision-making processes. Therefore, integrating a HITL mechanism requires careful design and implementation.

#### Triaging the Question

The core of this question lies in understanding how to handle low-confidence predictions in machine learning models deployed via FastAPI. Specifically:

What does "low confidence" mean? We need to define a threshold for model confidence.
What actions should be taken when confidence is low? This could involve returning an error message, suggesting further user input, or escalating the decision to a human expert.
How can we build an escalation path from the serving layer to a human expert? This involves designing both the technical and operational aspects of the system.

#### First Principles Solution

Define Confidence Thresholds:

Determine what constitutes "low confidence." For example, if your model's confidence level is below 80%, it could be considered low.

Model Output Handling:

If the model’s output meets or exceeds the confidence threshold, return the prediction directly to the user.
If the model’s output falls below the confidence threshold, decide on an appropriate response:
Return a generic error message indicating that the prediction is uncertain.
Suggest additional data input from the user (e.g., more images, text descriptions).
Escalate the decision to a human expert.

Human-in-the-Loop Mechanism:

Design a system where low-confidence predictions are automatically routed to a dedicated team or application that can manually review and provide a final decision.
Ensure that the escalation process is seamless, with minimal delay between receiving the prediction and making the final call.

Technical Implementation in FastAPI:

Use FastAPI’s routing features to define endpoints for both automated predictions and human expert interventions.
Implement middleware or custom exception handlers to detect low-confidence scenarios and trigger the appropriate response.

#### Example
Code
PYTHON
from fastapi import FastAPI, HTTPException

app = FastAPI()

class ModelOutput:
    def __init__(self, prediction: str, confidence: float):
        self.prediction = prediction
        self.confidence = confidence

@app.post("/predict")
async def predict(model_output: ModelOutput):
    if model_output.confidence >= 0.8:  # Example threshold
        return {"prediction": model_output.prediction}
    else:
        raise HTTPException(status_code=422, detail="Model prediction is uncertain.")

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, when implementing a HITL system in FastAPI, the key steps are to define confidence thresholds, handle low-confidence predictions appropriately, and design an efficient escalation path. Common mistakes include:

Overlooking the importance of clear communication with users about uncertain predictions.
Not providing sufficient support or guidance for human experts during the review process.
Failing to account for scalability issues as the system grows.

#### Follow-up
questions might explore how to handle edge cases (e.g., high-impact decisions vs. low-impact decisions), the use of real-time communication tools, and the integration of logging and monitoring to ensure the reliability and performance of the HITL mechanism.

**Code:**

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

class ModelOutput:
    def __init__(self, prediction: str, confidence: float):
        self.prediction = prediction
        self.confidence = confidence

@app.post("/predict")
async def predict(model_output: ModelOutput):
    if model_output.confidence >= 0.8:  # Example threshold
        return {"prediction": model_output.prediction}
    else:
        raise HTTPException(status_code=422, detail="Model prediction is uncertain.")
```

**Flow:**

1. User Request
2. Model Predicted Output
3. If Confidence >= 80%, ReturnPrediction
4. Else, Escalate to Human Review
5. Human Reviews and ProvidesFinal Decision
6. Final Decision Sent Back toUser

