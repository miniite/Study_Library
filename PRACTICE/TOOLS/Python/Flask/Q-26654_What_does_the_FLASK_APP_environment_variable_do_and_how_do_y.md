---
type: "practice"
difficulty: "Basic"
---

# What does the FLASK_APP environment variable do and how do you point it at your app?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1 · [↑ Configuration Management](<../../../../TOOLS/Python/Flask/Configuration_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a basic Flask application, FLASK_APP is an environment variable that points to your main Python file containing the entry point of your Flask application. By setting this environment variable, you can run and debug your application using the built-in development server provided by Flask.

#### Triage

The question is asking for an understanding of what the FLASK_APP environment variable does and how to properly configure it. The key points to cover include:

What FLASK_APP represents.
How setting this variable helps in running a Flask application.
Steps required to set up the variable.

#### First Principles Solution

The FLASK_APP environment variable is crucial for initializing a Flask application, especially when using command-line tools like flask run. Here’s how it works:

Environment Variable Setup: You need to define this environment variable in your terminal or shell before running any commands that depend on it.
File Path Configuration: The value of FLASK_APP should be the path to your main Python file, typically where you import your Flask app object. For instance, if you have a file named app.py, setting FLASK_APP=app.py tells Flask where to find your application entry point.
Running the Application: Once set up correctly, running commands like flask run will use this configuration to start the development server.
Steps to Set Up

To illustrate with a simple example:

Create a Basic Flask App:

PYTHON
# app.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run()

Set the Environment Variable:

In a Unix-like shell (e.g., bash):
BASH
export FLASK_APP=app.py
Or in Windows PowerShell:
POWERSHELL
$env:FLASK_APP = "app.py"

Run the Application:

BASH
flask run

#### Closing Summary
and Common Mistakes
Ensure you have specified the correct file path for FLASK_APP.
Remember to use the exact filename without any extensions or leading/trailing slashes.
The environment variable is case-sensitive, so ensure it matches your shell’s casing.

#### Common mistakes
include:

Incorrectly specifying the file name/path.
Not setting the environment variable before running Flask commands.
Forgetting to add .py in the filename if you are using a Unix-like shell.

#### Conclusion

By correctly setting FLASK_APP, you can efficiently manage and run your Flask application, leveraging tools like the built-in development server. Always double-check the file path to avoid common pitfalls.

