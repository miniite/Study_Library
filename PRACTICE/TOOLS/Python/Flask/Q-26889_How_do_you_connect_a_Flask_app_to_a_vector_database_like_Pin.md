---
type: "practice"
difficulty: "Intermediate"
---

# How do you connect a Flask app to a vector database like Pinecone or Chroma and reuse the connection across requests?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Flask-SQLAlchemy](<../../../../TOOLS/Python/Flask/Flask-SQLAlchemy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To integrate a Flask application with a vector database like Pinecone or Chroma, the primary goal is to establish a persistent connection that can be reused across multiple requests without re-creating it every time. This ensures efficient resource management and improved performance.

#### Triaging the Question

The question primarily revolves around setting up a Flask app to connect to a vector database (Pinecone or Chroma) in a way that maintains the connection throughout the application lifecycle. The key points being probed are: understanding of Flask's request handling, the concept of singleton patterns for resource management, and how to handle database connections efficiently.

#### Assumptions
You have a basic setup of a Flask app.
You understand the high-level functionality of Pinecone or Chroma vector databases.
The connection details such as API keys and environment variables are available and securely managed.

#### Solution
from First Principles:
Step 1: Setting Up Dependencies

First, ensure you have the necessary libraries installed:

BASH
pip install flask pinecone_client
Step 2: Configuring Environment Variables

Store your Pinecone or Chroma API keys in environment variables for security. You can set these using a .env file with python-dotenv or directly through Flask's configuration.

Step 3: Establishing the Connection

Use Python’s singleton pattern to ensure that the database connection is established only once and reused across requests.

PYTHON
from flask import Flask, request
import pinecone
from decouple import config

app = Flask(__name__)

class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            instance = super().__call__(*args, **kwargs)
            cls._instances[cls] = instance
        return cls._instances[cls]

class VectorDatabase(metaclass=SingletonMeta):
    def __init__(self):
        pinecone.init(api_key=config('PINECONE_API_KEY'))
        self.index_name = "my-vector-index"  # Replace with your index name

db_connection = VectorDatabase()

@app.route('/search', methods=['POST'])
def search():
    query_vector = request.json.get("query_vector")

    result = db_connection.index_name.query(vector=query_vector, top_k=5)
    return {"matches": [match.id for match in result.matches]}
Step 4: Testing the Connection

You can test this setup using a simple curl command or Postman to send a POST request with a query vector.

BASH
curl -X POST -H "Content-Type: application/json" -d '{"query_vector": [0.1, 0.2, 0.3, 0.4, 0.5]}' http://localhost:5000/search

#### Closing Summary

By using a singleton class to manage the database connection in Flask, you ensure that the connection is established only once and reused across multiple requests. This approach optimizes resource usage and enhances performance.

#### Common Mistakes/Follow-Ups
Mistake 1: Not using a singleton pattern can lead to unnecessary reconnections on each request, increasing latency.
Mistake 2: Forgetting to securely manage API keys in environment variables or hardcoded into the code.

#### Follow-Up
Discuss how this solution scales with more complex applications and multiple instances. Explore alternative connection management strategies like connection pooling.

**Code:**

```python
from flask import Flask, request
import pinecone
from decouple import config

app = Flask(__name__)

class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            instance = super().__call__(*args, **kwargs)
            cls._instances[cls] = instance
        return cls._instances[cls]

class VectorDatabase(metaclass=SingletonMeta):
    def __init__(self):
        pinecone.init(api_key=config('PINECONE_API_KEY'))
        self.index_name = "my-vector-index"  # Replace with your index name

db_connection = VectorDatabase()

@app.route('/search', methods=['POST'])
def search():
    query_vector = request.json.get("query_vector")

    result = db_connection.index_name.query(vector=query_vector, top_k=5)
    return {"matches": [match.id for match in result.matches]}
```

**Example input / output:**

```text
curl -X POST -H "Content-Type: application/json" -d '{"query_vector": [0.1, 0.2, 0.3, 0.4, 0.5]}' http://localhost:5000/search

# Expected Output:
{"matches": ["id1", "id2", "id3", "id4", "id5"]}
```

