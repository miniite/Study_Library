---
type: "practice"
difficulty: "Basic"
---

# How would you accept text in a FastAPI endpoint, compute its embedding, and return the vector?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ File Uploads and Inference](<../../../../TOOLS/Python/FastAPI/File_Uploads_and_Inference.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this FastAPI question, we need to understand how to set up an endpoint that accepts text input, processes it by computing its embedding using a pre-trained model (like BERT or Word2Vec), and then returns the resulting vector as JSON. We'll start with a brief overview of what's involved, describe the requirements, and walk through the implementation step-by-step.

FastAPI is a modern Python web framework that emphasizes performance and simplicity. In this scenario, we need to create an endpoint where users can input text, which will be processed by a machine learning model (like a pre-trained transformer model), and the resulting embedding vector will be returned as JSON.

First, let's define what we're solving:

We need an API that accepts a POST request with text data.
The text is then passed through a pre-trained embedding model.
Finally, the output vector from the model is returned in a JSON response.

#### Assumptions
for this solution include having access to a pre-trained machine learning model and understanding how to integrate it into our FastAPI application. We'll use transformers by Hugging Face for loading and using the model.

Step-by-Step Implementation

Install Required Libraries: Ensure you have FastAPI, Pydantic (for request validation), and Transformers library installed.

BASH
pip install fastapi uvicorn transformers pydantic

Create a Simple FastAPI Application: We'll start by setting up a basic FastAPI application and defining our endpoint.

Define the Input Model: Use Pydantic to define a model for the input data.

Load the Pre-trained Model: Use Hugging Face's Transformers library to load and use the pre-trained model.

Create the Endpoint: Define an endpoint that processes the text, computes its embedding, and returns the result as JSON.

Here’s the complete implementation:

PYTHON
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import torch
from transformers import AutoModel, AutoTokenizer

app = FastAPI()

# Load pre-trained model and tokenizer
model_name = "bert-base-uncased"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModel.from_pretrained(model_name)

class TextInput(BaseModel):
    text: str

@app.post("/embed")
async def embed_text(text_input: TextInput):
    # Tokenize the input text
    inputs = tokenizer([text_input.text], return_tensors="pt", padding=True, truncation=True)

    # Compute embeddings
    with torch.no_grad():
        outputs = model(**inputs)
        embedding_vector = outputs.last_hidden_state.mean(dim=1).squeeze().numpy()  # Average pooling

    return {"embedding": embedding_vector.tolist()}
Explanation

Import Libraries:

FastAPI and HTTPException for handling HTTP requests.
BaseModel from Pydantic to define the input schema.
AutoModel, AutoTokenizer from Transformers library.

Define Input Model:

A simple Pydantic model is defined to handle the text input.

Load Pre-trained Model:

Use Hugging Face’s AutoTokenizer and AutoModel classes to load the pre-trained BERT model.

Endpoint Definition:

Define a POST endpoint that accepts text input, tokenizes it, computes the embedding using average pooling over tokens, and returns the resulting vector as JSON.

#### Common Mistakes
and Follow-Ups
Mistake: Forgetting to handle edge cases where the input might be empty or too long. Always validate inputs.

#### Follow-Up
How would you handle more complex tokenization or embeddings? (e.g., using BERT's last hidden state directly, handling batches of texts).

#### Follow-Up
What about scaling this to a production environment?

