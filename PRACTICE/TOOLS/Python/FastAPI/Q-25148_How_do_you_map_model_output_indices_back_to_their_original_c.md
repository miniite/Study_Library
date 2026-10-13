---
type: "practice"
difficulty: "Basic"
---

# How do you map model output indices back to their original class names in a FastAPI serving app?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Model Consistency Across Deploys](<../../../../TOOLS/Python/FastAPI/Model_Consistency_Across_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To map model output indices back to their original class names in a FastAPI serving app, we need to ensure that our model's predictions are easily interpretable by mapping numerical indices from the model’s output to human-readable class labels. This is crucial for providing meaningful feedback to users or applications consuming the API.

#### Triaging the Question

The core of this question revolves around understanding how to associate numeric outputs with their corresponding class names in a FastAPI application. Essentially, we're looking at:

Model Output: The numerical index returned by our model's prediction.
Class Names: A mapping between these indices and their respective class labels.

#### Assumptions
We have a pre-trained model that outputs the class index of its predictions.
Our FastAPI application will receive this index as an output from the model.
The list of class names is known and fixed, such as ["cat", "dog", "bird"].

#### Solution
From First Principles
Define Class Labels: We need a way to store our class labels in a format that can be easily accessed by both the model and the API. A simple dictionary or array would suffice.
Mapping Mechanism: Create a mapping function that takes an index (output from the model) and returns the corresponding class name.

Here’s how we can implement this:

Step 1: Define Class Labels

Let's assume our classes are ["cat", "dog", "bird"]:

PYTHON
class_names = ["cat", "dog", "bird"]
Step 2: Create a Mapping Function

This function will take the model output index and return the corresponding class name.

PYTHON
def map_index_to_class(index):
    if 0 <= index < len(class_names):
        return class_names[index]
    else:
        return "Unknown"
Full Implementation in FastAPI

Now, we can integrate this mapping logic into a FastAPI endpoint. Here’s an example of how it might look:

PYTHON
from fastapi import FastAPI
import uvicorn

app = FastAPI()

# Define class labels
class_names = ["cat", "dog", "bird"]

@app.get("/predict")
async def predict(input_data: str):
    # Assume `input_data` is processed and model predicts an index
    predicted_index = 1  # Example prediction, replace with actual model logic

    result = map_index_to_class(predicted_index)

    return {"class": result}

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)

#### Closing Summary

In summary, to map model output indices back to their original class names in a FastAPI serving app, we need to:

Define the list of class labels.
Implement a mapping function that takes an index and returns the corresponding class name.
Integrate this logic into our FastAPI endpoint.

#### Common Mistakes/Follow-Ups
Misunderstanding Model Output: Not correctly interpreting the output from the model, leading to incorrect indices.
Inconsistent Class Names: Using different sets of class names in training and prediction phases.
Edge Cases Handling: Not handling cases where the predicted index is out of bounds.

By ensuring these points are addressed, we can provide accurate and meaningful responses from our FastAPI serving app.

