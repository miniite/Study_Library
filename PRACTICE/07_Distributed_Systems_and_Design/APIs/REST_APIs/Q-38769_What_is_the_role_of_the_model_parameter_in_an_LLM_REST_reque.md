---
type: "practice"
difficulty: "Basic"
---

# What is the role of the model parameter in an LLM REST request, and what happens if you send a model name the provider does not recognize?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Handling 429 Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Handling_429_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The model parameter in an LLM (Language Model) REST request specifies which model to use for processing the input text. Typically, this is done by passing the model's unique identifier or name as a query parameter or part of the URL. If you send a model name that the provider does not recognize, your request will likely fail with an error response.

This question touches on several key concepts: understanding REST API parameters, model identification in LLMs, and handling errors gracefully. Let’s break it down from first principles.

**Example input / output:**

```

Given an input request:

HTTP
Copy
POST https://api.example-llm.com/v1/engines/gpt-3.5-turbo/generate HTTP/1.1
Content-Type: application/json

{
  "prompt": "What is the weather like today?"
}

A correct response might be:

JSON
Copy
{
  "response": "The current weather conditions are sunny with a high of 72°F."
}

However, if you mistakenly use an unrecognized model name such as gpt-4, the API might respond with:

JSON
Copy
{
  "error": {
    "code": "MODEL_NOT_FOUND",
    "message": "The requested model 'gpt-4' is not supported by this service."
  }
}
```

