---
type: "practice"
difficulty: "Basic"
---

# How would you build a FastAPI endpoint that extracts named entities like names and dates from text using an LLM?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ LLM Endpoints in FastAPI](<../../../../TOOLS/Python/FastAPI/LLM_Endpoints_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a FastAPI endpoint that extracts named entities like names and dates from text using an LLM, we need to set up a few key components: setting up FastAPI, integrating with the LLM, and defining the API endpoints. The core idea is to use FastAPI's simplicity and flexibility to create a robust, RESTful service for entity extraction.

First, let’s break down the question:

What are we building? A FastAPI endpoint that processes text and uses an LLM (like Hugging Face's Transformers) to extract named entities.
What is FastAPI probing for? An understanding of how to integrate external services, manage API endpoints, and handle text processing.

We'll make some assumptions:

The entity extraction model is already available via a library like transformers from Hugging Face.
We’ll use an existing model trained for named entity recognition (NER).

Now let’s walk through the steps:

Install FastAPI and other dependencies:

BASH
pip install fastapi uvicorn

Create a simple FastAPI application:

PYTHON
# main.py
from fastapi import FastAPI, HTTPException, Request
import uvicorn
from transformers import pipeline

app = FastAPI()

# Load the NER model from Hugging Face
ner_model = pipeline("ner")

@app.post("/extract-entities")
async def extract_entities(request: Request):
    try:
        # Get the text from the request body
        data = await request.json()
        if "text" not in data:
            raise HTTPException(status_code=400, detail="Text is required")

        input_text = data["text"]
        entities = ner_model(input_text)

        return {"entities": entities}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
Run the FastAPI application:
BASH
uvicorn main:app --reload

This code sets up a simple FastAPI server with one endpoint /extract-entities that accepts POST requests containing text data and returns the extracted entities.

Test the API:

You can test this using curl or Postman:

BASH
# Example curl command
curl -X POST http://127.0.0.1:8000/extract-entities -H "Content-Type: application/json" -d '{"text": "John Doe was born on January 15, 1990."}'

Expected response:

JSON
{
    "entities": [
        {"word": "John", "start": 0, "end": 4, "entity": "PER"},
        {"word": "Doe", "start": 5, "end": 7, "entity": "PER"},
        {"word": "January", "start": 19, "end": 26, "entity": "DATE"},
        {"word": "15", "start": 28, "end": 30, "entity": "DATE"},
        {"word": "1990", "start": 32, "end": 37, "entity": "DATE"}
    ]
}

This setup ensures the endpoint is robust and can handle errors gracefully.

