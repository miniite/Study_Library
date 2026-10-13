---
type: "practice"
difficulty: "Basic"
---

# When an LLM API rejects your request with a 400, what are common reasons in the context of prompts, and how would you read the error body?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Handling 429 Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Handling_429_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with an LLM (Language Model) API like many modern AI services, if your request is rejected with a 400 Bad Request status code, it means there are issues with the payload or structure of your request. Common reasons include malformed prompts, incorrect data types, and exceeding allowed token limits. The error body will often provide clues about what exactly went wrong.

To effectively handle this situation, you need to understand that the API is rejecting the request because it doesn't meet its requirements. The 400 response typically includes a JSON-formatted error message in the body, which contains details such as validation errors or specific issues with your input data.

**Comparison:**

```
Common Error Type	Example Message	Explanation
length exceeded	Length of input exceeds the allowed limit.	The prompt or data field is too long for the API.
invalid format	Invalid value encountered in 'prompt'.	The input does not match the expected data format.
missing field	Field 'required_field' is missing from request body.	A required field was omitted from the request payload.
```

**Code:**

```python
# Example Python code snippet to handle a 400 response
import requests

response = requests.post("https://api.example.com/llm", json={"prompt": "This is an invalid prompt"})
if response.status_code == 400:
    error_body = response.json()
    print(f"Error: {error_body['message']}")
```

**Example input / output:**

```

For example, if you send a prompt that is too long:

JSON
Copy
POST https://api.example.com/llm
{
  "prompt": "This is an extremely long and repetitive string that goes far beyond the maximum token length allowed by the API."
}

Response:

JSON
Copy
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "code": 400,
  "message": "Bad Request",
  "errors": [
    {
      "field": "prompt",
      "message": "Length of input exceeds the allowed limit"
    }
  ]
}
```

**Data:**

```json
{
  "code": 400,
  "message": "Bad Request",
  "errors": [
    {
      "field": "prompt",
      "message": "Length of input exceeds the allowed limit"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Receive 400 Bad Request
4. Check status_code == 400?
5. Parse error body JSON
6. Extract and log specific fielderrors
7. Handle error gracefully, e.g., byre-trying with corrected inputor informing user
8. Continue processing therequest if no other issuesfound

