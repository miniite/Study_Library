---
type: "practice"
difficulty: "Basic"
---

# How do you create a minimal 'Hello World' Flask application?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1 · [↑ Flask vs Dedicated Model Servers](<../../../../TOOLS/Python/Flask/Flask_vs_Dedicated_Model_Servers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When creating a minimal "Hello World" Flask application, we are essentially setting up a basic web server that responds with the string "Hello, World!" when visited through a web browser or an HTTP client. This is a foundational example to understand how Flask works and can be extended for more complex applications.

Flask is a lightweight Python web framework designed to make getting started quick and easy, but still powerful enough to handle larger projects. It does not require any additional configuration or dependencies beyond the standard library when it comes to creating this simple application.

Here are the steps we will take:

Install Flask (if you haven't already).
Create a basic structure for our application.
Define and run the server.

Let's dive into each step in detail:

First, ensure that you have Python installed on your system. You can install Flask using pip by running pip install flask. For this example, we will assume you have both Python and Flask installed.

Next, let’s create our application:

Create a Directory for the Project: Start by creating a new directory for your project and navigate into it.
Create the Application File: Inside this directory, create a file named app.py. This is where we will write our Flask code.

Here's how you can do it step-by-step:

PYTHON
# Step 1: Import the Flask module
from flask import Flask

# Step 2: Create an instance of the Flask class. The first argument is usually __name__.
app = Flask(__name__)

# Step 3: Define a route and its corresponding handler function.
@app.route('/')
def hello_world():
    return 'Hello, World!'

# Step 4: Run the application if this script is executed directly
if __name__ == '__main__':
    app.run(debug=True)

In this code:

We import the Flask class from the flask module.
We create an instance of the Flask class, passing it the name of the current module (__name__).
We define a route for the root URL ('/') and associate it with the hello_world function, which returns the string "Hello, World!".
Finally, we check if the script is being run directly (not imported) and start the Flask development server with debug mode enabled.

To run your application, save this code in a file named app.py, go to the directory containing app.py, and execute it using the command:

SH
python app.py

If everything is set up correctly, you should see output similar to:

SH
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

Now, open a web browser and go to http://127.0.0.1:5000/. You should see "Hello, World!" displayed in the browser.

This setup is minimal and straightforward. However, it's important to note that for production use, you would need to consider more aspects like error handling, security, logging, and deployment strategies.

