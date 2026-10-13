---
type: "practice"
difficulty: "Deep"
---

# Explain shadow deployment for an ML model. How do you mirror production traffic to the new model, compare its predictions against the live one at scale, and avoid the shadow inference doubling your GPU cost or corrupting downstream state?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Mixed Version Fleets](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Shadow deployment is a critical technique in distributed systems, particularly important for ML models where you want to test a new version of the model without disrupting live traffic. The goal is to mirror production traffic to a new instance of the model, compare its predictions against those from the live model, and ensure that this process does not inadvertently increase costs or cause side effects.

To achieve shadow deployment effectively while avoiding common pitfalls, we need to break down the problem into several steps: setting up the infrastructure, managing resources efficiently, and ensuring fault tolerance. Let's go through these steps in detail.

Infrastructure Setup:

Model Serving: Set up a separate instance or container for the new model version.
Traffic Splitting: Use an ingress gateway that can split traffic between the live model and the shadow model based on configurable percentages (e.g., 90% to the live model, 10% to the shadow).
Monitoring & Logging: Implement logging and monitoring tools to track both models' outputs and any potential discrepancies.

Resource Management:

Cost Control: Avoid doubling your GPU costs by configuring the shadow deployment to use shared resources or dedicated but smaller instances.
State Consistency: Ensure that downstream systems (like databases or other services) do not inadvertently update based on the shadow model's outputs, which could lead to data corruption.

Predictions Comparison:

Inference Pipeline: Design an inference pipeline where both models run concurrently and their outputs are compared.
Validation Mechanism: Implement a validation mechanism that flags any discrepancies between the live and shadow models for further investigation.
Feedback Loop: Use the comparison results to fine-tune or discard the new model version before promoting it fully.

By adhering to these principles, we can effectively manage shadow deployments while minimizing risks and ensuring robust testing of new ML models.

**Comparison:**

```
Component	Description
Live Model	The current version serving production traffic
Shadow Model	A new version under test, running concurrently with the live model
Ingress Gateway	Manages traffic splitting and routes requests to appropriate models
Logging & Monitoring	Tracks both models' predictions and flags any discrepancies
```

**Code:**

```python
# Example Python code for traffic splitting using Flask (simplified)
from flask import Flask, request

app = Flask(__name__)

# Live model endpoint
@app.route('/live', methods=['POST'])
def live_model():
    data = request.json
    # Call the live model and return its prediction
    live_prediction = call_live_model(data)
    return {"prediction": live_prediction}

# Shadow model endpoint
@app.route('/shadow', methods=['POST'])
def shadow_model():
    data = request.json
    # Call the shadow model and return its prediction
    shadow_prediction = call_shadow_model(data)
    return {"prediction": shadow_prediction}

def call_live_model(data):
    # Code to call live model
    pass

def call_shadow_model(data):
    # Code to call shadow model
    pass

# Example traffic splitting logic (simplified)
import random

@app.route('/predict', methods=['POST'])
def predict():
    data = request.json
    if random.random() < 0.1:  # 10% chance for shadow deployment
        return app.shadow_model(data)
    else:
        return app.live_model(data)

# Example traffic configuration (simplified JSON)
traffic_config = {
    "live_traffic": 90,
    "shadow_traffic": 10
}
```

**Example input / output:**

```text
Input to both models (example JSON):
{"user_id": 123, "features": [1.5, 2.3, -0.7]}

Output of live model:
{"prediction": "high_risk"}

Output of shadow model:
{"prediction": "low_risk"}

Comparison result: Discrepancy detected (live vs. shadow)
```

**Data:**

```json
{
    "live_traffic_percentage": 90,
    "shadow_traffic_percentage": 10,
    "total_traffic": 100
}
```

**Flow:**

1. Live Model
2. (User Request)
3. Shadow Model
4. (Predictions)
5. Log Predictions
6. Monitor Discrepancies
7. Flag Issues

