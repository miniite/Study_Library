---
type: "practice"
difficulty: "Deep"
---

# How would you validate at CI time that a model artifact and the FastAPI serving code agree on feature names, order, and dtypes before the container is ever deployed?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Model Consistency Across Deploys](<../../../../TOOLS/Python/FastAPI/Model_Consistency_Across_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a machine learning model artifact and the corresponding FastAPI serving code are in sync regarding feature names, order, and data types, we can implement CI (Continuous Integration) checks during our development process. This ensures that any discrepancies between these two components are caught early on, preventing deployment issues.

First, we need to understand what exactly is being validated:

Feature Names: The set of feature names in the model should match those expected by the FastAPI server.
Order of Features: The order in which features are passed to the model must be the same as how they're handled by the API.
Data Types (dtypes): The data types of input features should align between the model and the API.

Next, we will break down the solution step-by-step from first principles:

Step 1: Model Artifact Inspection

The model artifact could be in various formats like pickle, joblib, or custom serialization. Regardless of the format, you can extract the relevant information about feature names and dtypes during deserialization. For instance, if using joblib, you might inspect the metadata stored within.

Step 2: FastAPI API Inspection

In the FastAPI code, we typically define the input schema for our API endpoints. This is usually done via a Pydantic model or through type hints. We can extract this information programmatically to ensure it matches the expected feature names and dtypes.

Step 3: CI Integration

We integrate these checks into our CI pipeline using tools like GitHub Actions, Jenkins, or GitLab CI/CD. Here’s how we would implement these steps in a real-world scenario:

Code Example

First, let's look at an example of extracting feature information from the model artifact:

PYTHON
# Example: Extracting feature names and dtypes from a joblib model
import joblib

def extract_model_features(model_path):
    # Load the model
    model = joblib.load(model_path)

    # Assuming the model has a method to get feature names
    feature_names = model.feature_names

    # Get data types (dtypes) of features from the model metadata
    dtypes = [type(value).__name__ for value in next(iter(model.predict([feature_names])))]

    return feature_names, dtypes

# Example: Extracting schema information from FastAPI Pydantic models
from pydantic import BaseModel

class InputSchema(BaseModel):
    age: int
    income: float
    education_level: str

def extract_api_features(schema_class):
    # Get the model fields and their types
    feature_names = [field.name for field in schema_class.__fields__.values()]
    dtypes = [str(field.annotation) for field in schema_class.__fields__.values()]

    return feature_names, dtypes
CI Integration Example

We can now integrate these checks into our CI pipeline:

YAML
# GitHub Actions Example
name: Model Validation

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  validate-model-artifact:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.8'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt

      - name: Validate model artifact
        run: |
          # Call the functions to validate features and dtypes
          model_features, model_dtypes = extract_model_features('path/to/model.pkl')
          api_features, api_dtypes = extract_api_features(InputSchema)

          if model_features != api_features or model_dtypes != api_dtypes:
            echo "Error: Model artifact and API schema do not match!"
            exit 1

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the key steps are to programmatically extract feature information from both the model artifact and the FastAPI serving code. By integrating these checks into our CI pipeline, we can ensure that any discrepancies are caught early on, reducing the risk of deployment issues.

#### Common mistakes
include:

Ignoring Feature Order: Ensuring not only names but also order matches.
Data Type Mismatch: Verifying dtypes precisely to avoid unexpected errors at runtime.
Manual Validation: Not automating these checks can lead to human error and missed discrepancies.

#### Follow-ups
might involve considering additional validation steps, such as checking for null or invalid values in the input data, or exploring more sophisticated model export formats that support richer metadata.

**Code:**

```python
# Example code snippets
def extract_model_features(model_path):
    # Load the model
    model = joblib.load(model_path)

    # Assuming the model has a method to get feature names
    feature_names = model.feature_names

    # Get data types (dtypes) of features from the model metadata
    dtypes = [type(value).__name__ for value in next(iter(model.predict([feature_names])))]

    return feature_names, dtypes

class InputSchema(BaseModel):
    age: int
    income: float
    education_level: str

def extract_api_features(schema_class):
    # Get the model fields and their types
    feature_names = [field.name for field in schema_class.__fields__.values()]
    dtypes = [str(field.annotation) for field in schema_class.__fields__.values()]

    return feature_names, dtypes
```

