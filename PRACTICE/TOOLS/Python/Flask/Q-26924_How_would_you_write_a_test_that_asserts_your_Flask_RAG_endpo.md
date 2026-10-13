---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a test that asserts your Flask RAG endpoint includes the retrieved sources in its response?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ RAG Endpoints in Flask](<../../../../TOOLS/Python/Flask/RAG_Endpoints_in_Flask.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To test if a Flask RESTful API endpoint that uses Retrieval-Augmented Generation (RAG) returns the correct sources, we need to ensure our test covers both the retrieval and generation parts of the RAG process. This involves setting up an environment where we can mock the source retrieval logic and verify that it is included in the generated response.

#### Triage

The question asks for a test case to validate that a Flask endpoint returning responses from a Retrieval-Augmented Generation (RAG) system includes the retrieved sources. The key aspects here are:

Mocking the retrieval part of RAG.
Ensuring the retrieved sources are included in the generated response.

#### Assumptions
We have a basic understanding of how the RAG endpoint works, including both the retrieval and generation components.
We use Python's unittest or pytest for our test cases.
The Flask app has an endpoint /generate that takes some input (e.g., user query) and returns a response containing sources.

#### First-Principles Solution

To achieve this, we will:

Set up a simple Flask application with the RAG endpoint.
Mock the retrieval logic using Python's unittest.mock.
Write test cases to assert that the retrieved sources are included in the response.

Here’s a step-by-step breakdown:

Step 1: Setup Flask Application

First, let's define our basic Flask app and endpoint.

PYTHON
from flask import Flask, jsonify

app = Flask(__name__)

def retrieve_sources(query):
    # Simulate source retrieval logic (for simplicity)
    return ["source1", "source2"]

@app.route('/generate', methods=['POST'])
def generate():
    query = request.json['query']
    sources = retrieve_sources(query)
    response_content = f"Generated content using {len(sources)} sources."
    return jsonify({
        'response': response_content,
        'sources': sources
    })
Step 2: Mock the Retrieval Logic

We need to mock retrieve_sources to provide predictable data.

PYTHON
from unittest.mock import patch, MagicMock

def test_generate_endpoint():
    # Mocking the retrieve_sources function
    with patch('app.retrieve_sources') as mock_retrieve_sources:
        # Define expected sources and response content
        mock_retrieve_sources.return_value = ["source1", "source2"]
        expected_response_content = {
            'response': 'Generated content using 2 sources.',
            'sources': ['source1', 'source2']
        }

        with app.test_client() as client:
            # Make a POST request to the /generate endpoint
            response = client.post('/generate', json={'query': 'example query'})

            # Assert that the correct data was returned in the response
            assert response.status_code == 200, "HTTP status code is not 200"
            assert response.json['sources'] == expected_response_content['sources'], "Sources are missing or incorrect"

# This function would be part of a test suite or file.
Step 3: Run and Validate the Test

Finally, we run this test to ensure everything works as expected. If mock_retrieve_sources returns the correct sources, our test should pass.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key takeaway is that by mocking the retrieval logic, we can control the data flow through our application and verify its behavior. Common mistakes include not properly setting up the mock or not correctly asserting the expected response content.

For follow-ups:

#### Cons
ider edge cases where no sources are found.
Ensure other parts of the RAG process (like generation) are also tested thoroughly.
Use more sophisticated mocking techniques as needed, such as responses library for HTTP requests.

**Code:**

```

from flask import Flask, jsonify

app = Flask(name)

def retrieve_sources(query): return ["source1", "source2"]

@app.route('/generate', methods=['POST']) def generate(): query = request.json['query'] sources = retrieve_sources(query) response_content = f"Generated content using {len(sources)} sources." return jsonify({ 'response': response_content, 'sources': sources })

CODE
Copy

```python
from unittest.mock import patch, MagicMock

def test_generate_endpoint():
    with patch('app.retrieve_sources') as mock_retrieve_sources:
        mock_retrieve_sources.return_value = ["source1", "source2"]
        expected_response_content = {
            'response': 'Generated content using 2 sources.',
            'sources': ['source1', 'source2']
        }

        with app.test_client() as client:
            response = client.post('/generate', json={'query': 'example query'})

            assert response.status_code == 200, "HTTP status code is not 200"
            assert response.json['sources'] == expected_response_content['sources'], "Sources are missing or incorrect"
```

