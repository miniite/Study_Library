---
type: "practice"
difficulty: "Basic"
---

# How do you accept free text input and run it through an NLP model in a FastAPI endpoint?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ File Uploads and Inference](<../../../../TOOLS/Python/FastAPI/File_Uploads_and_Inference.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To accept free text input and run it through an NLP model using a FastAPI endpoint, we need to follow several steps from first principles. This involves setting up the FastAPI server, defining the endpoint that accepts text inputs, preprocessing the text data, running it through the NLP model, and then returning the processed output. Let's break down each step in detail.

First, let's quickly triage what this question is probing:

Understanding of FastAPI: This includes how to set up a basic FastAPI server.
Handling Input Data: How to receive text input from HTTP requests and validate it.
NLP Preprocessing: Basic steps for preprocessing the text data (like tokenization, stop words removal).
Running NLP Model: How to pass the preprocessed text through an NLP model.
Returning Output: Formatting the response to return meaningful results.

Now, let's walk through each step:

Setting Up FastAPI Server:

We will start by creating a basic setup for our FastAPI application.

Defining Endpoint to Accept Text Input:

We need to define an endpoint that accepts text data via HTTP POST requests.

Preprocessing the Text Data:

Preprocess the incoming text data (e.g., tokenization, stop words removal).

Running NLP Model on Processed Text:

Use a pre-trained or custom-built NLP model to process the cleaned text.

Returning the Output:

Format and return the processed result in a structured response.
Step-by-Step Solution

Setting Up FastAPI Server:

PYTHON
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class InputText(BaseModel):
    text: str

@app.post("/process_text/")
async def process_text(input_data: InputText):
    return {"message": "Received and will process your input."}

Defining Endpoint to Accept Text Input:

We use a POST endpoint at /process_text/ that expects an object containing the text data.

Preprocessing the Text Data:

Preprocessing can be done using libraries like NLTK or spaCy.
PYTHON
import nltk
from nltk.corpus import stopwords

def preprocess(text):
    # Tokenize and remove stop words
    tokens = nltk.word_tokenize(text)
    cleaned_tokens = [token for token in tokens if token not in stopwords.words('english')]
    return ' '.join(cleaned_tokens)

Running NLP Model on Processed Text:

Assume we have a simple model that classifies text into categories.
PYTHON
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.naive_bayes import MultinomialNB

# Example model setup
vectorizer = CountVectorizer()
model = MultinomialNB()

def predict_category(text):
    cleaned_text = preprocess(text)
    features = vectorizer.transform([cleaned_text])
    category = model.predict(features)[0]
    return {"category": category}

Returning the Output:

The endpoint will call the preprocessing and prediction functions, then return a structured response.
PYTHON
@app.post("/process_text/")
async def process_text(input_data: InputText):
    text = input_data.text
    cleaned_text = preprocess(text)
    category = predict_category(cleaned_text)
    return {"category": category["category"]}

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution covers the basic steps to accept free text input, process it using an NLP model in a FastAPI endpoint. Key points include correctly setting up the FastAPI server, handling input data, preprocessing text, running through a simple NLP model, and returning meaningful results.

#### Common Mistakes

Not properly validating input data.
Not handling edge cases (e.g., empty strings).
Failing to preprocess the text data adequately before feeding it into the model.
Returning complex objects without proper structure.

For follow-ups:

What if we need to handle multiple categories?
How would you optimize this endpoint for performance?

**Code:**

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class InputText(BaseModel):
    text: str

@app.post("/process_text/")
async def process_text(input_data: InputText):
    return {"message": "Received and will process your input."}

import nltk
from nltk.corpus import stopwords

def preprocess(text):
    tokens = nltk.word_tokenize(text)
    cleaned_tokens = [token for token in tokens if token not in stopwords.words('english')]
    return ' '.join(cleaned_tokens)

from sklearn.feature_extraction.text import CountVectorizer
from sklearn.naive_bayes import MultinomialNB

vectorizer = CountVectorizer()
model = MultinomialNB()

def predict_category(text):
    cleaned_text = preprocess(text)
    features = vectorizer.transform([cleaned_text])
    category = model.predict(features)[0]
    return {"category": category}

@app.post("/process_text/")
async def process_text(input_data: InputText):
    text = input_data.text
    cleaned_text = preprocess(text)
    category = predict_category(cleaned_text)
    return {"category": category["category"]}
```

**Example input / output:**

```text
Input:
{
  "text": "This is a sample sentence for testing the NLP model."
}

Output:
{
  "category": "sample_category"
}
```

**Flow:**

1. mermaid
2. graph TD
3. A[Receive Text Input] --> B{Validate Input?}
4. B -- Yes --> C[Preprocess Text]
5. C --> D[Run NLP Model]
6. D --> E[Predict Category]
7. E --> F[Return Result]
8. B -- No --> G[Error Handling]

