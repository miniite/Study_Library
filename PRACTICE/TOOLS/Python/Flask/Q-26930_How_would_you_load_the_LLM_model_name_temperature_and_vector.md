---
type: "practice"
difficulty: "Basic"
---

# How would you load the LLM model name, temperature, and vector-store settings from environment variables in a Flask GenAI app?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Configuration Management](<../../../../TOOLS/Python/Flask/Configuration_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To configure an AI Engineer Fresher on how to load the LLM (Large Language Model) model name, temperature, and vector-store settings from environment variables in a Flask GenAI application, we need to understand that these configurations will be stored securely outside the codebase. This approach enhances security by keeping sensitive information like API keys and specific parameters separate from our source code.

#### Triaging the Question

The question is essentially asking for how to read configuration values directly from the environment variables in a Flask web application. The interviewer might be probing your understanding of:

Environment Variables: How they are used to store configurations.
Flask Configuration Handling: Using app.config or similar mechanisms.
Security Practices: Ensuring sensitive data is not hardcoded.
Step-by-Step Solution
1. Setting Up Environment Variables

First, we need to set up the environment variables in a .env file (using tools like python-dotenv):

PLAINTEXT
MODEL_NAME=llama-2-7b-hf
TEMPERATURE=0.9
VECTOR_STORE_PATH=/path/to/vector-store
2. Loading Environment Variables into Flask App

We'll use the python-dotenv package to load these environment variables:

Install python-dotenv:

BASH
pip install python-dotenv

Create a .env file and populate it with your settings.

Load the environment variables in your Flask app:

PYTHON
from flask import Flask
from dotenv import load_dotenv

app = Flask(__name__)

# Load environment variables from .env file
load_dotenv()

# Accessing these values as configuration
model_name = os.getenv('MODEL_NAME')
temperature = float(os.getenv('TEMPERATURE'))
vector_store_path = os.getenv('VECTOR_STORE_PATH')

@app.route('/')
def home():
    return f"Model Name: {model_name}, Temperature: {temperature}, Vector Store Path: {vector_store_path}"

if __name__ == '__main__':
    app.run()
3. Running the Flask App

Ensure your environment variables are correctly set up and then run the Flask application:

BASH
export MODEL_NAME=llama-2-7b-hf
export TEMPERATURE=0.9
export VECTOR_STORE_PATH=/path/to/vector-store

python app.py

#### Closing Summary
+ Common Mistakes/Follow-Ups

In this solution, we have demonstrated how to securely load configuration settings from environment variables in a Flask application using python-dotenv. This approach keeps sensitive information out of the source code and ensures that configurations can be easily changed without modifying the application code. Common mistakes include:

Forgetting to install python-dotenv or misconfiguring it.
Hardcoding sensitive values directly into the application, which poses a security risk.

#### Follow-ups
might involve discussing alternative ways of managing environment variables (like using Docker .env files) and how to handle different deployment environments securely.

**Code:**

```python
from flask import Flask
from dotenv import load_dotenv

app = Flask(__name__)

# Load environment variables from .env file
load_dotenv()

# Accessing these values as configuration
model_name = os.getenv('MODEL_NAME')
temperature = float(os.getenv('TEMPERATURE'))
vector_store_path = os.getenv('VECTOR_STORE_PATH')

@app.route('/')
def home():
    return f"Model Name: {model_name}, Temperature: {temperature}, Vector Store Path: {vector_store_path}"

if __name__ == '__main__':
    app.run()
```

**Data:**

```plaintext
MODEL_NAME=llama-2-7b-hf
TEMPERATURE=0.9
VECTOR_STORE_PATH=/path/to/vector-store
```

