---
type: "practice"
difficulty: "Basic"
---

# How would you document the input and output schema of your ML prediction endpoint using FastAPI's automatic docs?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pydantic Validation](<../../../../TOOLS/Python/FastAPI/Pydantic_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To document the input and output schemas of an ML prediction endpoint in FastAPI, we need to ensure that our API is both clear and self-explanatory. This involves defining the structure of the data being passed into and out of our endpoints using Pydantic models. Here’s a step-by-step guide on how to achieve this:

FastAPI provides auto-generated interactive documentation via Swagger UI or ReDoc, which makes it easy for developers (and other stakeholders) to understand the API without writing extensive comments.

**Code:**

```python
from fastapi import FastAPI
from pydantic import BaseModel

# Define a Pydantic model for the input schema
class InputData(BaseModel):
    feature1: float
    feature2: int
    feature3: str

# Define a Pydantic model for the output schema
class OutputData(BaseModel):
    prediction: float

app = FastAPI()

@app.post("/predict")
def predict(input_data: InputData) -> OutputData:
    # Simulate an ML model prediction
    prediction = 0.5 * input_data.feature1 + 0.2 * input_data.feature2 - 0.3 * len(input_data.feature3)

    return {"prediction": prediction}
```

**Example input / output:**

```

The InputData and OutputData models define the expected input and output formats:

Example Input:

JSON
Copy
{
    "feature1": 3.5,
    "feature2": 7,
    "feature3": "example"
}

Expected Output:

JSON
Copy
{
    "prediction": 0.82
}
```

**Flow:**

1. No flowchart is needed as the logic is straightforward and implemented directly in the code.

