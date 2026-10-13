---
type: "practice"
difficulty: "Basic"
---

# What is request.values in Flask and what does it combine?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1 · [↑ Request Parsing and Validation](<../../../../TOOLS/Python/Flask/Request_Parsing_and_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Flask, request.values is a powerful attribute that combines multiple sources of data from HTTP requests into a single object. This allows developers to easily access form data, query string parameters, and files sent in multipart/form-data without needing to manually check which source the data came from. The key idea here is that request.values provides a unified way to handle request input.

To triage the question: You are being asked about the functionality of request.values, specifically its role in combining different types of inputs. The interviewer might probe your understanding of Flask's request handling and how this attribute simplifies common tasks for web developers.

First, let’s understand from first principles:

In a typical HTTP request, data can come from three main sources: form fields (POST or PUT requests), query strings (URL parameters in GET requests), and uploaded files.
Flask's request object has several attributes to handle these inputs separately. For example, request.form for form data, request.args for URL query string parameters, and request.files for file uploads.

However, when you need a unified way to access all of this data together, that’s where request.values comes in handy. Here’s how it works step by step:

Initialization: When Flask receives a request, the request object is initialized with various attributes based on the type and content of the request.
Combining Data Sources: request.values internally combines data from request.form, request.args, and request.files. It creates an immutable MultiDict that allows you to access keys as if they were a dictionary, but it maintains all the combined values.
PYTHON
from flask import Flask, request

app = Flask(__name__)

@app.route('/form', methods=['POST'])
def handle_form():
    # Accessing form data and query string parameters through request.values
    name = request.values.get('name')
    age = request.values.get('age')
    city = request.values.get('city')

    return f"Name: {name}, Age: {age}, City: {city}"

In this example, request.values allows you to access both form data (name, age) and query string parameters (city). If a key exists in multiple sources, it will be present only once in the combined MultiDict.

#### Closing Summary

request.values is a convenient way to handle mixed input from forms and URLs in Flask. It simplifies accessing all kinds of request data without needing to check individual attributes like form, args, or files. However, keep in mind that if you need mutable behavior, you should use the underlying dictionaries directly.

#### Common Mistakes
and Follow-Ups
Mistake: Using request.values as a mutable object instead of understanding it as an immutable MultiDict.

#### Follow-Up
Discuss how to handle cases where data might come from multiple sources simultaneously. For example, if you need to ensure that only one source’s value takes precedence over another.

**Code:**

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/form', methods=['POST'])
def handle_form():
    name = request.values.get('name')
    age = request.values.get('age')
    city = request.values.get('city')

    return f"Name: {name}, Age: {age}, City: {city}"
```

