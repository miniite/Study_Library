---
type: "practice"
difficulty: "Intermediate"
---

# How would you evaluate whether your FastAPI RAG endpoint gives good answers, and what simple metrics could a fresher track?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ RAG Endpoints in FastAPI](<../../../../TOOLS/Python/FastAPI/RAG_Endpoints_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To evaluate whether a FastAPI REST API for Retrieval-Augmented Generation (RAG) provides good answers, a fresher should focus on both qualitative and quantitative aspects. The key is to set up simple metrics that can be tracked and analyzed over time. Here’s how you would approach this:

Firstly, the RAG endpoint needs to provide accurate and relevant responses to user queries. A quick way to start evaluating this is by tracking response accuracy through a few basic metrics.

**Code:**

```python
# Example of a simple response metric calculation in FastAPI

from fastapi import FastAPI
import json

app = FastAPI()

@app.post("/evaluate_response")
async def evaluate_response(query: str, generated_response: str):
    # Load the expected answer from a predefined JSON file or database
    with open("expected_answers.json", "r") as f:
        expected_answers = json.load(f)

    user_query = query.lower()
    if user_query in expected_answers:
        expected_answer = expected_answers[user_query]["answer"]

        # Check for exact match, partial match, or any other custom logic
        if generated_response == expected_answer:
            return {"accuracy": 1.0}
        elif expected_answer in generated_response:
            return {"accuracy": 0.8}  # Partial match scenario
        else:
            return {"accuracy": 0.5}  # No match scenario
    else:
        return {"accuracy": 0.0}  # Query not found in the dataset

This code snippet demonstrates a simple endpoint that evaluates the generated response against expected answers stored in expected_answers.json.

Secondly, you can track user satisfaction and engagement by collecting feedback through an optional user rating system. This involves adding a frontend form where users can rate the quality of responses.
```

**Data:**

```json
{
  "user_ratings": {
    "query1": {"rating": 4},
    "query2": {"rating": 3},
    "query3": {"rating": 5}
  }
}
```

