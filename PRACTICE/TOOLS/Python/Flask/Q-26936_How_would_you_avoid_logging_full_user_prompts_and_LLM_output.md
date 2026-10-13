---
type: "practice"
difficulty: "Deep"
---

# How would you avoid logging full user prompts and LLM outputs in a Flask app when they may contain personal data?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Streaming Responses](<../../../../TOOLS/Python/Flask/Streaming_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of avoiding logging full user prompts and LLM (Language Model) outputs that may contain personal data in a Flask application, we need to ensure that sensitive information is not stored or logged directly. This involves designing a solution that anonymizes or masks sensitive parts while still allowing relevant data for debugging purposes.

#### Triage

The core of the question revolves around ensuring privacy and security by handling user prompts and LLM outputs carefully. The interviewer might be probing into various aspects such as data masking techniques, logging policies, and best practices to balance functionality with privacy.

#### First-Principles Solution
Identify Sensitive Information: Determine what parts of the user prompt or LLM output should be masked.
Masking Strategy: Implement a strategy for anonymizing or replacing sensitive information.
Logging Policy: Define which logs are necessary and which can be safely discarded.
Audit Trails: Maintain minimal logging that still allows tracking of events without compromising data.
Step-by-Step Solution

Identify Sensitive Information:

Common sensitive information includes names, addresses, financial details, etc.
Use regular expressions or pattern matching to identify these elements in the user prompt and LLM outputs.

Masking Strategy:

Replace sensitive words with a generic placeholder like [REDACTED].
For longer strings (like credit card numbers), truncate them while maintaining some structure.

Logging Policy:

Log only metadata such as timestamps, user IDs, and log levels.

#### Cons
ider using differential logging where more detailed logs are created for specific cases (e.g., errors) but not stored in persistent storage.

Audit Trails:

Maintain a minimal audit trail that includes enough information to track the flow of data without exposing sensitive details.

#### Implementation

Here’s an example implementation using Flask:

PYTHON
from flask import Flask, request, jsonify
import re

app = Flask(__name__)

def mask_sensitive_data(text):
    # Example regex for matching names and financial data
    mask_patterns = [
        (r'\b[A-Z][A-Za-z]+\b', '[REDACTED_NAME]'),
        (r'\d{16}', '[REDACTED_CC_NUMBER]')
    ]

    for pattern, replacement in mask_patterns:
        text = re.sub(pattern, replacement, text)
    return text

@app.route('/api/llm', methods=['POST'])
def process_llm_request():
    data = request.json
    user_prompt = data.get('prompt')
    llm_output = data.get('output')

    # Mask sensitive information
    masked_user_prompt = mask_sensitive_data(user_prompt)
    masked_llm_output = mask_sensitive_data(llm_output)

    log_message = f"Prompt: {masked_user_prompt}\nOutput: {masked_llm_output}"

    # Log to console or a secure logging service
    print(log_message)

    response = {"processed": True, "prompt": user_prompt, "output": llm_output}
    return jsonify(response)

if __name__ == '__main__':
    app.run(debug=True)

#### Common Mistakes
and Follow-Ups

Not Masking Sensitive Information:

Simply logging everything without any masking can expose sensitive data.

Incomplete Masking Strategy:

Failing to define clear patterns for what constitutes sensitive information.

Over-Logging:

Keeping too much detailed log data, which can be a privacy risk and also waste storage space.

Not Using Differential Logging:

Failing to use differential logging where more detailed logs are created only under specific conditions (e.g., errors).

Debugging Issues:

Over-masking might lead to issues during debugging, so ensure that critical information is still available for troubleshooting.

#### Conclusion

By carefully designing a masking strategy and implementing it in the Flask application, you can effectively handle sensitive data without compromising privacy or functionality. Always balance between security and maintainability by adhering to best practices like differential logging and minimal logging policies.

**Code:**

```python
from flask import Flask, request, jsonify
import re

app = Flask(__name__)

def mask_sensitive_data(text):
    # Example regex for matching names and financial data
    mask_patterns = [
        (r'\b[A-Z][A-Za-z]+\b', '[REDACTED_NAME]'),
        (r'\d{16}', '[REDACTED_CC_NUMBER]')
    ]

    for pattern, replacement in mask_patterns:
        text = re.sub(pattern, replacement, text)
    return text

@app.route('/api/llm', methods=['POST'])
def process_llm_request():
    data = request.json
    user_prompt = data.get('prompt')
    llm_output = data.get('output')

    # Mask sensitive information
    masked_user_prompt = mask_sensitive_data(user_prompt)
    masked_llm_output = mask_sensitive_data(llm_output)

    log_message = f"Prompt: {masked_user_prompt}\nOutput: {masked_llm_output}"

    # Log to console or a secure logging service
    print(log_message)

    response = {"processed": True, "prompt": user_prompt, "output": llm_output}
    return jsonify(response)

if __name__ == '__main__':
    app.run(debug=True)
```

