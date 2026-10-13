---
type: "practice"
difficulty: "Intermediate"
---

# How do you handle a query parameter that can accept multiple values, e.g. tags? Compare repeated keys versus comma-separated values.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Filtering Sorting and Pagination](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Filtering_Sorting_and_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with query parameters in REST APIs that can accept multiple values, such as tags, there are two common approaches: using repeated key-value pairs or sending a single value as a comma-separated string. In this answer, I will explain both methods from first principles, discuss their pros and cons, and provide practical examples.

#### Triage

The question is asking you to compare two ways of handling multiple values for the same query parameter in REST APIs:

Repeated Key-Value Pairs: The API treats each occurrence of a key as an independent value.
Comma-Separated Values: A single string containing all values separated by commas.

#### First Principles Solution
Repeated Key-Value Pairs

When you use repeated keys, the client sends multiple instances of the same parameter in the query string. This approach is more flexible and adheres to standard HTTP/URI conventions.

HTTP
GET /api/items?tags=technology&tags=machine-learning&tags=data-science

The server can easily parse these as individual values without any special handling.

Comma-Separated Values

Alternatively, the client might send a single query parameter with multiple values concatenated into a comma-separated string.

HTTP
GET /api/items?tags=technology,machine-learning,data-science

The server then needs to handle this string and split it into individual tags. This approach is more compact but requires additional logic on the server side.

#### Triage
Recap and Assumptions
The API supports both methods for flexibility.
We need to compare these in terms of readability, maintainability, security, and potential issues like injection attacks or handling edge cases.

#### Cons
ider a scenario where you might have thousands of tags, which could impact performance.

#### Solution
Handling Repeated Key-Value Pairs

When using repeated keys:

HTTP Request:
HTTP
GET /api/items?tags=technology&tags=machine-learning&tags=data-science
Server-Side Parsing:
Use a framework’s built-in support for query parameters or manually parse the request.

#### Example
in Python Flask:
PYTHON
from flask import request

@app.route('/api/items')
def items():
    tags = request.args.getlist('tags')
    # Process tags list
    return {'tags': tags}
Handling Comma-Separated Values

When using a single comma-separated value:

HTTP Request:
HTTP
GET /api/items?tags=technology,machine-learning,data-science
Server-Side Parsing:
Split the string on commas.

#### Example
in Python Flask:
PYTHON
from flask import request

@app.route('/api/items')
def items():
    tags_str = request.args.get('tags', '')
    tags = [tag.strip() for tag in tags_str.split(',')]
    # Process tags list
    return {'tags': tags}

#### Closing Summary
and Common Mistakes/Follow-Ups

Handling query parameters that can accept multiple values is a common requirement. Both repeated keys and comma-separated values have their merits and pitfalls.

Repeating Keys:

#### Pros
Easier to implement, no special parsing needed.

#### Cons
Potentially long URLs for many tags, not standard in all frameworks.

Comma-Separated Values:

#### Pros
Shorter URLs, more compact.

#### Cons
Requires additional string splitting logic on the server side, potential edge cases like trailing commas.

Additional considerations include:

Security: Ensure proper input validation to prevent injection attacks.
Performance: Handling large numbers of tags efficiently is crucial.
User Experience: Consider how tags are displayed and used in the UI.

#### Follow-up
questions might involve discussing performance implications or exploring alternatives such as using an array in JSON bodies for complex APIs.

