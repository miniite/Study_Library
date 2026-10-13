---
type: "practice"
difficulty: "Expert"
---

# A principal probe: your platform's Python pickled sklearn models break on framework upgrades and can't be served on non-Python runtimes. How would you drive an org-wide move to a portable serialization (ONNX/native) and manage the long tail that resists?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Offline Evaluation Harnesses](<../../../../TOOLS/Python/Python_Language/Safe_User-authored_Configuration/Offline_Evaluation_Harnesses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of sklearn models breaking with Python framework upgrades, we need to ensure a seamless migration path to more portable serialization formats like ONNX or native code. This involves several steps: identifying and prioritizing critical models, developing a strategy for converting them, and managing the transition in an iterative manner.

The core challenge here is ensuring that our machine learning models remain usable across different runtimes without requiring extensive rework each time we upgrade or switch frameworks. Let's break down the process step by step.

First, it’s crucial to understand what portable serialization means. Portable serialization formats like ONNX (Open Neural Network Exchange) allow models to be represented in a format that is language-agnostic and platform-independent. This makes them easier to deploy across different environments without needing extensive rework on every upgrade.

Next, we need to assess the current state of our models:

Identify Critical Models: Evaluate which models are most critical for business operations.
Prioritize Migration: Develop a priority list based on model complexity and usage frequency.

Now, let's dive into the technical aspects:

Step 1: Assess Current Models

We need to gather information about each model:

Model type (e.g., decision tree, neural network)
Version of sklearn used
Performance metrics

This can be done by iterating through our current model storage and collecting this metadata.

PYTHON
import os
from joblib import load
from sklearn.utils.estimator_checks import check_estimator

# Example function to collect model information
def get_model_info(model_path):
    try:
        estimator = load(model_path)
        version = check_estimator(estimator).version
        performance_metrics = calculate_performance_metrics(estimator)  # Custom function to fetch metrics
        return {
            'path': model_path,
            'type': type(estimator).__name__,
            'version': version,
            'metrics': performance_metrics
        }
    except Exception as e:
        print(f"Error reading {model_path}: {e}")
        return None

# Collect all models in a directory
models = [get_model_info(os.path.join(model_dir, f)) for f in os.listdir(model_dir)]
Step 2: Convert Models to ONNX

For sklearn models that support ONNX conversion, we can use the skl2onnx package. This requires converting each model to an ONNX representation.

PYTHON
from skl2onnx import convert_sklearn
import onnx

def convert_model_to_onnx(model_path, output_path):
    try:
        estimator = load(model_path)
        onx = convert_sklearn(estimator)  # Convert using skl2onnx
        with open(output_path, 'wb') as f:
            f.write(onx.SerializeToString())
        return True
    except Exception as e:
        print(f"Failed to convert {model_path}: {e}")
        return False

# Example usage
converted = [convert_model_to_onnx(model['path'], model_dir + '/' + os.path.basename(model['path']).replace('.joblib', '.onnx')) for model in models]
Step 3: Handle Non-Portable Models

For models that don't support ONNX conversion, we need to look at alternative solutions:

Re-implement from Scratch: If the model is simple and has a clear implementation, re-writing it in another language might be feasible.
Use Native Code: For more complex models, generating native code using tools like TVM or PyTorch’s ONNX export can help.
Step 4: Deployment Strategy

Deploy the new portable models alongside the old ones initially to ensure no downtime and easy rollback if issues arise.

PYTHON
from fastapi import FastAPI
import uvicorn

app = FastAPI()

@app.get("/predict")
async def predict(model_type: str, data: dict):
    model_path = f"{model_dir}/{model_type}.onnx"  # Assuming ONNX models are in this directory
    if not os.path.exists(model_path):
        return {"error": "Model not found"}
    else:
        onx = onnx.load(model_path)
        # Load and run the model (simplified example)
        result = run_model(onx, data)
        return {"prediction": result}

# Example prediction function
def run_model(onx, data):
    session = InferenceSession(model_path)
    inputs = {name: np.array(data[name]) for name in session.get_inputs()[0].type.name}
    return session.run(None, inputs)[0][0]

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)

#### Closing Summary
and Common Mistakes

To summarize, the key steps are:

Identify and prioritize critical models.
Convert to ONNX or native code using tools like skl2onnx.
Develop a deployment strategy to ensure smooth transition.

#### Common mistakes
include over-reliance on manual re-implementation for complex models, underestimating the effort required for conversion, and not having a robust rollback plan in place.

**Comparison:**

```
Model Type	Version	Metrics
RandomForestClassifier	0.24.2	accuracy: 0.85, f1_score: 0.79
GradientBoostingRegressor	0.23.2	r2_score: 0.88, mse: 15.4
```

**Code:**

```python
import os
from joblib import load
from sklearn.utils.estimator_checks import check_estimator

def get_model_info(model_path):
    try:
        estimator = load(model_path)
        version = check_estimator(estimator).version
        performance_metrics = calculate_performance_metrics(estimator)  # Custom function to fetch metrics
        return {
            'path': model_path,
            'type': type(estimator).__name__,
            'version': version,
            'metrics': performance_metrics
        }
    except Exception as e:
        print(f"Error reading {model_path}: {e}")
        return None

models = [get_model_info(os.path.join(model_dir, f)) for f in os.listdir(model_dir)]

def convert_model_to_onnx(model_path, output_path):
    try:
        estimator = load(model_path)
        onx = convert_sklearn(estimator)
        with open(output_path, 'wb') as f:
            f.write(onx.SerializeToString())
        return True
    except Exception as e:
        print(f"Failed to convert {model_path}: {e}")
        return False

converted = [convert_model_to_onnx(model['path'], model_dir + '/' + os.path.basename(model['path']).replace('.joblib', '.onnx')) for model in models]

from fastapi import FastAPI
import uvicorn

app = FastAPI()

@app.get("/predict")
async def predict(model_type: str, data: dict):
    model_path = f"{model_dir}/{model_type}.onnx"
    if not os.path.exists(model_path):
        return {"error": "Model not found"}
    else:
        onx = onnx.load(model_path)
        result = run_model(onx, data)
        return {"prediction": result}

def run_model(onx, data):
    session = InferenceSession(model_path)
    inputs = {name: np.array(data[name]) for name in session.get_inputs()[0].type.name}
    return session.run(None, inputs)[0][0]

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

**Example input / output:**

```text
Input:
GET /predict?model_type=RandomForestClassifier&data={"feature1": 1.5, "feature2": 3.2}

Output:
{"prediction": 0.987654}
```

**Data:**

```json
{
  "model_dir": "/path/to/models",
  "models": [
    {
      "path": "/path/to/models/model1.joblib",
      "type": "RandomForestClassifier",
      "version": "0.24.2",
      "metrics": {"accuracy": 0.85, "f1_score": 0.79}
    },
    {
      "path": "/path/to/models/model2.joblib",
      "type": "GradientBoostingRegressor",
      "version": "0.23.2",
      "metrics": {"r2_score": 0.88, "mse": 15.4}
    }
  ]
}
```

**Flow:**

1. Identify Critical Models
2. Prioritize
3. Convert to ONNX
4. Deploy New Models
5. Rollback Plan
6. Monitor Performance
7. Maintain Models

