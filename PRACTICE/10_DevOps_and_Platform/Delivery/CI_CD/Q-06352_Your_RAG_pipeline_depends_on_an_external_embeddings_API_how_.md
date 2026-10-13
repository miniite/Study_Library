---
type: "practice"
difficulty: "Intermediate"
---

# Your RAG pipeline depends on an external embeddings API; how would you run integration tests in CI without racking up API cost or being blocked when the provider is down?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Ephemeral Environments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Ephemeral_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, our goal is to ensure that the RAG (Retrieval-Augmented Generation) pipeline's embeddings functionality works correctly without incurring unnecessary API costs during CI/CD runs or when the external service is temporarily down. This involves setting up a local mock server for testing and configuring environment variables to switch between real and mocked APIs.

To achieve this, we need to:

Mock the external embeddings API.
Configure the application to use the mocks in CI environments.
Ensure minimal overhead or false positives during integration tests.

Firstly, let's define what we mean by "mocking" — it involves creating a local server that mimics the behavior of the real API but allows us to control its responses and interactions without relying on external network calls. This is crucial for reducing costs and ensuring reliable testing environments.

**Code:**

```python
# Example setup in Python using Flask for mocking an embeddings API

from flask import Flask, jsonify
import os

app = Flask(__name__)

@app.route('/embeddings/<string:input_text>', methods=['GET'])
def get_embedding(input_text):
    # Here we can define a mapping or generate dummy embeddings
    embedding_response = {
        "embedding": [0.1, 0.2, 0.3],
        "input_text": input_text
    }
    return jsonify(embedding_response)

if __name__ == '__main__':
    app.run(port=int(os.getenv('EMBEDDINGS_API_PORT', 5000)))

This is a simple Flask application that serves as an embeddings API and can be started locally. The environment variable EMBEDDINGS_API_PORT allows us to specify the port on which it runs.
```

**Example input / output:**

```

To demonstrate how we can switch between real and mock APIs in our RAG pipeline, consider the following Python code snippet using requests:

PYTHON
Copy
import os
import requests

def get_embedding(input_text):
    api_url = os.getenv('EMBEDDINGS_API_URL', 'http://real-embeddings-api.com/embeddings')  # Default to real API

    try:
        response = requests.get(f"{api_url}/{input_text}")
        return response.json()
    except requests.exceptions.RequestException as e:
        print(f"Failed to fetch embeddings: {e}")
        # Fallback to local mock server
        if os.getenv('USE_LOCAL_EMBEDDINGS', 'false') == 'true':
            from local_embeddings import get_embedding  # Import the mocked function
            return get_embedding(input_text)

This code first tries to hit the real API using EMBEDDINGS_API_URL. If a network request fails, it falls back to the locally running mock server.
```

**Data:**

```yaml
# Example .env file for setting up local development or CI environments

EMBEDDINGS_API_URL=http://localhost:5000  # URL of the mocked embeddings API

The .env file contains the necessary environment variables to configure our application. In a CI/CD setup, this file would be replaced with values specific to the testing environment.
```

