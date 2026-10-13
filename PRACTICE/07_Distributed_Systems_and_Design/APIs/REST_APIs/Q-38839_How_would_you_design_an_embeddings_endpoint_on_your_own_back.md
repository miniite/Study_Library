---
type: "practice"
difficulty: "Intermediate"
---

# How would you design an /embeddings endpoint on your own backend that batches many texts and returns their vectors efficiently?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Vector Search APIs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Vector_Search_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an efficient /embeddings endpoint, we need to ensure that the system can handle multiple text inputs in a batch, process them using a pre-trained model or custom embedding algorithm, and return vector representations efficiently. The core idea is to optimize both memory usage and computational efficiency while maintaining high accuracy.

#### Triaging the Question

The interviewer is probing your understanding of:

Batch Processing: How to handle multiple text inputs at once.
Efficiency: Minimizing computational resources (time, memory) used per request.
Scalability: Ensuring the endpoint can scale with increasing numbers of texts or embedding dimensions.

#### First-Principles Solution
Input Handling:
Accept a JSON array of text inputs via POST to /embeddings.
Model Inference:
Use an efficient model for batch inference.
Batch Processing:
Minimize the number of inferences by batching similar texts together.
Response Formatting:
Return vectors as a JSON object with input text and corresponding vector.
Step-by-Step Solution

Input Handling

PYTHON
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/embeddings', methods=['POST'])
def embeddings():
    texts = request.json.get('texts')
    if not texts:
        return jsonify({"error": "Missing text inputs"}), 400

    # Placeholder for model inference
    vectors = batch_infer(texts)

    response = [{"text": text, "vector": vector} for text, vector in zip(texts, vectors)]
    return jsonify(response)

Model Inference

Use a pre-trained embedding model like BERT or custom embeddings.
Ensure the model is optimized for batch inference.

Batch Processing

Tokenization: Split texts into tokens and handle them in batches.
Padding: Pad shorter texts to ensure consistent input size.
Parallel Processing: Use multiple CPU cores or GPU threads for parallel inference.

Response Formatting

PYTHON
def batch_infer(texts):
    # Example using a pre-trained model (placeholder)
    from transformers import pipeline

    nlp = pipeline("feature-extraction", model="bert-base-uncased")
    vectors = [v[0] for v in nlp(texts, padding=True, truncation=True, return_tensors=True)]
    return vectors

#### Closing Summary
and Common Mistakes/Follow-Ups

To design this endpoint efficiently:

Optimize Tokenization: Use efficient tokenizers like Hugging Face’s AutoTokenizer.
Batching Strategy: Optimize batch size to balance memory usage and model performance.
Model Selection: Choose a lightweight but accurate model for production.
Error Handling: Implement proper error handling, especially for input validation.

#### Common mistakes
include:

Failing to use batching, leading to inefficient single-text processing.
Not optimizing the number of inferences or parallelism.
Overlooking memory constraints with large batch sizes.

#### Follow-Ups
How would you handle very large batches?
What strategies would you use to optimize memory usage?
How do you balance model accuracy and inference speed?

**Code:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/embeddings', methods=['POST'])
def embeddings():
    texts = request.json.get('texts')
    if not texts:
        return jsonify({"error": "Missing text inputs"}), 400

    vectors = batch_infer(texts)
    response = [{"text": text, "vector": vector} for text, vector in zip(texts, vectors)]
    return jsonify(response)

def batch_infer(texts):
    from transformers import pipeline

    nlp = pipeline("feature-extraction", model="bert-base-uncased")
    vectors = [v[0] for v in nlp(texts, padding=True, truncation=True, return_tensors=True)]
    return vectors
```

**Example input / output:**

```text
POST /embeddings
Content-Type: application/json

{
  "texts": ["Hello world", "This is a test"]
}

Response:
[
  {"text": "Hello world", "vector": [0.1, 0.2, 0.3]},
  {"text": "This is a test", "vector": [-0.1, -0.2, -0.3]}
]
```

**Flow:**

1. Receive Request
2. Validate Inputs
3. Batch Texts
4. Perform Inference
5. Return Results

