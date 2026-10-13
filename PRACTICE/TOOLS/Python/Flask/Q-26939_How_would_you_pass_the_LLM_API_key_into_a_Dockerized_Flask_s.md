---
type: "practice"
difficulty: "Intermediate"
---

# How would you pass the LLM API key into a Dockerized Flask service securely at runtime rather than baking it into the image?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Configuration Management](<../../../../TOOLS/Python/Flask/Configuration_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To securely pass an LLM API key to a Dockerized Flask service at runtime, we need to avoid hardcoding sensitive information like API keys directly in the application code or Dockerfile. This is critical for security reasons and adheres to best practices such as using environment variables. Here’s how you can achieve this:

First, understand that the core idea involves using environment variables to securely pass sensitive data at runtime. Then, we will explore different methods to set these environment variables within a Docker container.

**Code:**

```
app.py - Flask application code

from flask import Flask, jsonify

app = Flask(name)

@app.route('/api') def get_api_key(): api_key = os.environ.get('LLM_API_KEY') # Accessing the API key from environment variable if not api_key: return jsonify({"error": "API Key is missing"}), 400 response_data = {"api_key": api_key} return jsonify(response_data)

if name == 'main': app.run()

CODE
Copy

```bash
# Dockerfile snippet to set environment variables
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

ENV LLM_API_KEY=your_secret_api_key_here  # This should be overridden at runtime

CMD ["python", "app.py"]
```

**Example input / output:**

```

To see how this works, let's consider the following input:

BASH
Copy
docker run -e LLM_API_KEY="your_secret_api_key_here" your_flask_image

The output would be a Flask API endpoint responding with the provided API key:

JSON
Copy
{
  "api_key": "your_secret_api_key_here"
}
```

