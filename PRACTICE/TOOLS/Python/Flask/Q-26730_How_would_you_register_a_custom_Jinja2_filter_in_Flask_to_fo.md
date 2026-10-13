---
type: "practice"
difficulty: "Intermediate"
---

# How would you register a custom Jinja2 filter in Flask to format currency, and use it in a template?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Routing and Blueprints](<../../../../TOOLS/Python/Flask/Routing_and_Blueprints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To register a custom Jinja2 filter in Flask for formatting currency, we need to follow these steps:

Define the filter function.
Register the filter with the Jinja2 environment used by the Flask app.
Use the newly registered filter in our templates.

Here's how you can achieve this from first principles:

TL;DR

Register a custom Jinja2 filter currency_format to format currency amounts and use it in your Flask templates.

#### Triaging the Question

The question is asking us to integrate a custom formatting function into our Flask application so that we can easily format monetary values within our HTML templates. The interviewer may be probing your understanding of Flask's template engine, Jinja2, and how to extend its capabilities by adding custom filters.

Step-by-Step Solution
Define the Filter Function First, we need a function that takes an amount as input and formats it according to our desired currency format (e.g., $X,XXX.00).
PYTHON
def currency_format(value):
    if value is None:
        return ''
    # Assuming we want US dollars with two decimal places
    return f"${value:,.2f}"
Register the Filter in Flask To register this filter, we need to access the Jinja2 environment used by Flask and add our custom function.
PYTHON
from flask import Flask

app = Flask(__name__)

# Registering the currency_format filter
@app.template_filter('currency')
def currency_format(value):
    if value is None:
        return ''
    # Formatting the value with US dollar symbol and commas
    return f"${value:,.2f}"

if __name__ == '__main__':
    app.run()
Use the Filter in Templates Now that our filter is registered, we can use it directly within any template.
HTML
<!-- templates/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Currency Formatting</title>
</head>
<body>
    <p>Total Amount: {{ 123456 | currency }}</p>
</body>
</html>

#### Closing Summary
and Common Mistakes

To summarize, we defined a simple currency_format function to format monetary values. We then registered this function as a custom Jinja2 filter within our Flask application. Finally, we demonstrated its usage in an HTML template.

#### Common mistakes
include forgetting to check for None values (which can cause errors), not registering the filter correctly, or misusing it in templates without understanding how to pass parameters and return formatted strings.

Self-Check

Correctness + completeness confirmed.

**Code:**

```python
from flask import Flask

app = Flask(__name__)

@app.template_filter('currency')
def currency_format(value):
    if value is None:
        return ''
    return f"${value:,.2f}"

if __name__ == '__main__':
    app.run()
```

**Example input / output:**

```text
<!-- templates/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Currency Formatting</title>
</head>
<body>
    <p>Total Amount: {{ 123456 | currency }}</p>
</body>
</html>
```

