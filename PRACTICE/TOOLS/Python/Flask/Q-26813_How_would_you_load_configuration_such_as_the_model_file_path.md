---
type: "practice"
difficulty: "Basic"
---

# How would you load configuration such as the model file path and threshold from environment variables in a Flask ML app?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Configuration Management](<../../../../TOOLS/Python/Flask/Configuration_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To load configuration settings like model file paths and thresholds from environment variables in a Flask ML application, we need to ensure that our code is flexible and can adapt to different environments easily. This involves setting up the environment variables on your development or production machine and fetching them within the Flask app.

Firstly, let's understand what we're probing:

We are looking at how you manage configuration settings dynamically.
You should demonstrate knowledge of environment variables in Python and Flask.
Understanding of a basic workflow for integrating external configurations into a Flask application.

We will assume that your project structure is typical, with the main Flask app file being app.py, and you might have additional files or directories to hold your models. The configuration settings such as model paths and thresholds could be stored in environment variables named, e.g., MODEL_PATH and THRESHOLD.

Step-by-Step Solution

Set Environment Variables: Before running the Flask application, set these environment variables on your system.

SH
export MODEL_PATH=/path/to/your/model.h5
export THRESHOLD=0.5

Accessing Environment Variables in Flask:

Import the os module to interact with the operating system and retrieve environment variables.

Fetching Configuration in Your App:

PYTHON
from flask import Flask
import os

app = Flask(__name__)

# Load model path from ENV variable
model_path = os.environ.get('MODEL_PATH')

if not model_path:
    raise ValueError("Model path must be set as an environment variable")

# Load threshold from ENV variable
threshold = float(os.environ.get('THRESHOLD', 0.5))

Using the Configurations in Your ML Logic:

You can now use model_path and threshold in your machine learning logic, like loading a model or applying some decision-making process based on the threshold.

#### Example
Code

Here’s an example of how you might integrate this into a basic Flask application:

