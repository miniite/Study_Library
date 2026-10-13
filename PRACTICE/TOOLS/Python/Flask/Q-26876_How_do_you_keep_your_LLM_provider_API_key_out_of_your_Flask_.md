---
type: "practice"
difficulty: "Basic"
---

# How do you keep your LLM provider API key out of your Flask code and load it from an environment variable instead?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Configuration Management](<../../../../TOOLS/Python/Flask/Configuration_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To keep your LLM (Language Model) provider API key out of your Flask code, we'll use environment variables. This is a common best practice in web development to ensure that sensitive information like API keys are not hardcoded into your source code. By loading the API key from an environment variable, you can better manage secrets and adhere to security practices.

#### Triaging the Question

The question essentially asks how to securely store and retrieve an API key for an LLM provider using Flask's configuration mechanisms. The interviewer is probing knowledge of secure coding practices, environment variable management, and understanding of Flask's flexibility in handling configuration.

Step-by-Step Solution

Install the required package: First, ensure you have python-dotenv installed to manage environment variables easily.

BASH
pip install python-dotenv

Store the API Key in an Environment Variable:

Set up a .env file in your project directory and add the following line:
CODE
LLM_API_KEY=your_secret_api_key_here

Load the Environment Variables in Flask Application:

Use python-dotenv to load these variables into your Flask application.
PYTHON
from flask import Flask
from dotenv import load_dotenv

app = Flask(__name__)

# Load environment variables from .env file
load_dotenv()

# Access the API key using os.getenv()
api_key = os.getenv('LLM_API_KEY')

@app.route('/')
def index():
    return f"API Key: {api_key}"

if __name__ == '__main__':
    app.run(debug=True)

#### Closing Summary
and Common Mistakes

By storing the API key in an environment variable, we ensure that sensitive information is not exposed directly in our codebase. This approach also allows for easier management of secrets, such as using a secrets manager or version control systems with access controls.

#### Common Mistakes

Hardcoding the API key directly into the source code.
Forgetting to load the environment variables before accessing them.
Not setting up proper security practices around managing and updating sensitive information.

