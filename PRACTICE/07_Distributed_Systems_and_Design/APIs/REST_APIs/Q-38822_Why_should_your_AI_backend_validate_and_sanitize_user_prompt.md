---
type: "practice"
difficulty: "Intermediate"
---

# Why should your AI backend validate and sanitize user prompts before forwarding them to an LLM API, even for a simple chatbot?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Keeping Secrets out of URLs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Keeping_Secrets_out_of_URLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Validating and sanitizing user prompts in an AI backend is crucial to ensure data integrity, prevent security vulnerabilities, and maintain the reliability of interactions with language models (LLMs). This process helps in filtering out malicious or inappropriate inputs, protecting sensitive information, and ensuring that only clean, structured data reaches the LLM API.

#### Triaging the Question

The question revolves around understanding why it's important to validate and sanitize user prompts before forwarding them to an LLM API. At its core, this involves ensuring data quality and security by filtering out harmful or invalid inputs. The interviewer is probing your knowledge of best practices in backend development, particularly in relation to handling untrusted input.

#### First-Principles Solution
Data Integrity: Ensuring that the user prompts are well-formed and contain no syntax errors can help prevent issues like crashes or unexpected behavior in the application.
Security: Validating inputs is essential to mitigate common security risks such as SQL injection, cross-site scripting (XSS), and other types of attacks.

#### Cons
istency: Sanitizing user inputs helps maintain consistency across different parts of your system by ensuring that data adheres to a specific format or standard.
LLM API Reliability: By sanitizing user prompts before sending them to the LLM, you can avoid passing malformed or inappropriate requests which could lead to unexpected responses from the model.
Step-by-Step Reasoning:
Initial Validation:
Input Parsing: The first step is to parse the input to ensure it follows a specific format (e.g., valid JSON for API calls).
Schema Checking: Validate the structure of the input against predefined schemas or patterns using tools like JSON Schema validation.
Sanitization:
Removing Malicious Content: Strip out any potentially harmful characters, such as <script>, ;--, or any other HTML tags that could be used for XSS attacks.
URL Encoding: Encode special characters to ensure they are interpreted correctly by the backend and LLM API.
Error Handling:
Handle validation errors gracefully by providing clear error messages and logging them for debugging purposes.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In summary, validating and sanitizing user prompts is a fundamental practice that helps in maintaining robustness and security of your AI application. This process ensures that only valid, clean data reaches the LLM API, thereby preventing issues such as crashes, unexpected behavior, or security breaches.

#### Common Mistakes

Failing to validate input formats can lead to unhandled exceptions.
Not sanitizing user inputs leaves the application vulnerable to XSS and SQL injection attacks.
Overlooking consistency checks might result in inconsistent data handling across different components of the system.

#### Follow-Ups

How do you handle edge cases where valid but unusual prompts are provided?
What measures do you take to log and report validation failures?

**Code:**

```python
import jsonschema
from flask import request, jsonify

# Define a schema for the user prompt
user_prompt_schema = {
    "type": "object",
    "properties": {
        "message": {"type": "string"},
        "metadata": {"type": "object"}
    },
    "required": ["message"]
}

def validate_user_prompt(prompt):
    try:
        # Parse and validate the input against the schema
        jsonschema.validate(instance=prompt, schema=user_prompt_schema)
        return True
    except jsonschema.exceptions.ValidationError as e:
        print(f"Invalid prompt: {e}")
        return False

@app.route('/api/chat', methods=['POST'])
def chat():
    if not request.is_json:
        return jsonify({"error": "Request must be JSON"}), 415

    user_prompt = request.get_json()

    # Validate the user prompt
    if validate_user_prompt(user_prompt):
        # Sanitize and clean up the input (e.g., URL encoding)
        sanitized_message = user_prompt['message'].replace('<', '&lt;').replace('>', '&gt;')

        # Forward to LLM API
        llm_response = call_llm_api(sanitized_message)
        return jsonify({"response": llm_response})
    else:
        return jsonify({"error": "Invalid prompt"}), 400
```

**Example input / output:**

```text
# Sample Input
{
    "message": "<script>alert('XSS')</script>",
    "metadata": {}
}

# Sample Output (After Validation and Sanitization)
{
    "response": "Sanitized response from LLM"
}
```

**Flow:**

1. Invalid
2. Valid
3. Receive User Prompt
4. Valid?
5. Log Error
6. Sanitize Input
7. Forward to LLM API
8. Handle Response

