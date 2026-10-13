---
type: "practice"
difficulty: "Deep"
---

# How do you handle a FastAPI model endpoint whose feature preprocessing depends on a lookup table that itself changes over time, keeping the table version pinned to the model?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Service Architecture and Versioning](<../../../../TOOLS/Python/FastAPI/Service_Architecture_and_Versioning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of handling a FastAPI model endpoint where feature preprocessing relies on a lookup table that evolves with time, we need to ensure that the correct version of the lookup table is used by the model at any given deployment. This involves managing multiple versions of the lookup table and ensuring they are seamlessly integrated into the FastAPI model pipeline. The key idea here is to decouple the lookup table from the model so that changes can be made independently without disrupting the service.

First, let's clarify what the question is really asking: We need a robust system for versioning the lookup tables and serving them dynamically based on the model’s deployment state or user configuration. This involves understanding the current state of the FastAPI application, managing multiple versions of the lookup table, and ensuring that these changes are propagated seamlessly to the model during inference.

#### Assumptions

The lookup table is stored in a version-controlled manner (e.g., CSV, JSON).
We have access to the version information when deploying or running the model.
The FastAPI application can be configured with the necessary parameters to select the appropriate lookup table version.

Now, let's reason through this from first principles:

Model and Lookup Table Versioning:

Maintain a directory structure that stores different versions of the lookup tables (e.g., lookup_tables/0.1.csv, lookup_tables/0.2.csv).
Use environment variables or configuration files to specify which version should be used during model inference.

FastAPI Integration:

Create an endpoint in FastAPI that loads the appropriate lookup table based on the provided version.
Implement a function within the preprocessing pipeline of the FastAPI model that reads and processes this lookup table dynamically.

Configuration Management:

Use a configuration management system (e.g., pydantic for Python) to handle environment variables or configuration files.
Ensure these configurations are read at runtime and reflect the correct version.

Deployment Strategy:

Automate the deployment process using CI/CD pipelines that can be triggered when a new lookup table is released.
Use tools like Docker or Kubernetes to manage the lifecycle of the FastAPI service, ensuring that only the relevant versions are served.

Testing and Validation:

Implement unit tests for the preprocessing pipeline to validate that it correctly loads and uses the appropriate version of the lookup table.
Validate the overall system by running end-to-end tests with different versions of the lookup table.

Here is a step-by-step solution:

Step 1: Directory Structure

Organize your lookup tables in a version-controlled directory:

PLAINTEXT
project/
├── app/
│   ├── main.py
│   └── preprocessing/
│       └── __init__.py
├── config/
│   └── settings.yml
└── lookup_tables/
    ├── 0.1.csv
    └── 0.2.csv
Step 2: Configuration Management

Use config/settings.yml to specify the version:

YAML
version: "0.2"
Step 3: FastAPI Model Endpoint

Define a FastAPI endpoint that loads and processes the lookup table:

PYTHON
# main.py
from fastapi import FastAPI, Depends
import pandas as pd
from app.preprocessing import get_lookup_table

app = FastAPI()

def load_lookup_table(version):
    return pd.read_csv(f'lookup_tables/{version}.csv')

@app.post("/predict")
async def predict(data: dict, version: str = Depends(get_lookup_table)):
    lookup_df = version(data)
    # Preprocess data using the lookup table
    processed_data = preprocess_data(data, lookup_df)
    # Make prediction with model
    prediction = model.predict(processed_data)
    return {"prediction": prediction}

def get_lookup_table(version):
    def inner(data):
        lookup_df = load_lookup_table(version)
        # Example: Perform preprocessing based on the lookup table
        return lookup_df.apply(lambda x: data.get(x, 0))
    return inner
Step 4: Preprocessing Pipeline

Implement the preprocess_data function:

PYTHON
# app/preprocessing/__init__.py
import pandas as pd

def preprocess_data(data, lookup_df):
    # Example preprocessing logic
    processed_data = pd.DataFrame([data])
    for col in lookup_df.columns:
        if col in data:
            processed_data[col] = data[col]
        else:
            processed_data[col] = 0
    return processed_data
Step 5: Deployment and CI/CD

Automate deployment using Docker and Kubernetes to ensure the correct version of the lookup table is used.

This solution ensures that changes to the lookup table do not affect the operational stability of the FastAPI model endpoint, allowing for smooth updates and maintenance.

