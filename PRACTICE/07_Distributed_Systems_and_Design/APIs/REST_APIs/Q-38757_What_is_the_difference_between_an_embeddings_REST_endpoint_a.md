---
type: "practice"
difficulty: "Basic"
---

# What is the difference between an embeddings REST endpoint and a chat completion REST endpoint in terms of what you send and what you get back?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Vector Search APIs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Vector_Search_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this answer, we'll explore the differences between an embeddings REST endpoint and a chat completion REST endpoint by focusing on what data they receive (input) and return (output). Both endpoints are part of API frameworks like OpenAI's API but serve different purposes.

#### Triage
and Core Idea

The question probes your understanding of how APIs process requests and generate responses. Specifically, it asks you to differentiate between two types of REST endpoints: one that provides embeddings and another for generating chat completions. Embeddings endpoints return numerical representations of text or other data, while chat completion endpoints produce human-like text based on the input provided.

Step-by-Step Solution

Embeddings Endpoint

Input: The endpoint receives a string (text) as input.
PLAINTEXT
POST /v1/embeddings HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "input": "Natural language processing is the field of study that focuses on the interactions between computers and human language."
}
Output: It returns an array of numerical values (embeddings) representing the input text.
JSON
{
  "object": "list",
  "data": [
    {
      "index": 0,
      "embedding": [0.123, -0.456, 0.789]
    }
  ],
  "usage": {
    "prompt_tokens": 15,
    "total_tokens": 15
  }
}

Chat Completion Endpoint

Input: The endpoint receives a prompt (string) and optionally parameters like temperature for controlling randomness.
PLAINTEXT
POST /v1/chat/completions HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "model": "text-davinci-003",
  "prompt": "Translate the following sentence to Spanish: 'The quick brown fox jumps over the lazy dog.'"
}
Output: It returns a string of human-like text as the response.
JSON
{
  "id": "cmpl-FooBar123",
  "object": "text_completion",
  "created": 1672598078,
  "model": "text-davinci-003",
  "choices": [
    {
      "index": 0,
      "text": "El rápido zorro marrón salta sobre el perro perezoso.",
      "logprobs": null,
      "finish_reason": "length"
    }
  ]
}

#### Closing Summary
and Common Mistakes

In summary, embeddings endpoints return numerical representations (embeddings) of input text, useful for tasks like similarity searches. Chat completion endpoints generate human-like text based on the provided prompts, making them ideal for applications requiring conversational AI.

#### Common mistakes
include confusing the purposes of these two types of endpoints or misunderstanding their outputs. For example, expecting chat completions to return numerical values instead of text can lead to miscommunication during development and integration phases.

**Comparison:**

| Embeddings Endpoint | Chat Completion Endpoint |
|---------------------|-------------------------|
| Receives text       | Receives prompt         |
| Returns embeddings  | Returns human-like text |

**Code:**

```

// Example request for embeddings endpoint POST /v1/embeddings HTTP/1.1 Host: api.example.com Content-Type: application/json

{ "input": "Natural language processing is the field of study that focuses on the interactions between computers and human language." }

CODE
Copy

```plaintext
// Example response from embeddings endpoint
{
  "object": "list",
  "data": [
    {
      "index": 0,
      "embedding": [0.123, -0.456, 0.789]
    }
  ],
  "usage": {
    "prompt_tokens": 15,
    "total_tokens": 15
  }
}
PLAINTEXT
Copy
// Example request for chat completion endpoint
POST /v1/chat/completions HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "model": "text-davinci-003",
  "prompt": "Translate the following sentence to Spanish: 'The quick brown fox jumps over the lazy dog.'"
}
PLAINTEXT
Copy
// Example response from chat completion endpoint
{
  "id": "cmpl-FooBar123",
  "object": "text_completion",
  "created": 1672598078,
  "model": "text-davinci-003",
  "choices": [
    {
      "index": 0,
      "text": "El rápido zorro marrón salta sobre el perro perezoso.",
      "logprobs": null,
      "finish_reason": "length"
    }
  ]
}
```

**Example input / output:**

```text
Input for Embeddings Endpoint:
POST /v1/embeddings HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "input": "Natural language processing is the field of study that focuses on the interactions between computers and human language."
}

Output from Embeddings Endpoint:
{
  "object": "list",
  "data": [
    {
      "index": 0,
      "embedding": [0.123, -0.456, 0.789]
    }
  ],
  "usage": {
    "prompt_tokens": 15,
    "total_tokens": 15
  }
}

Input for Chat Completion Endpoint:
POST /v1/chat/completions HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "model": "text-davinci-003",
  "prompt": "Translate the following sentence to Spanish: 'The quick brown fox jumps over the lazy dog.'"
}

Output from Chat Completion Endpoint:
{
  "id": "cmpl-FooBar123",
  "object": "text_completion",
  "created": 1672598078,
  "model": "text-davinci-003",
  "choices": [
    {
      "index": 0,
      "text": "El rápido zorro marrón salta sobre el perro perezoso.",
      "logprobs": null,
      "finish_reason": "length"
    }
  ]
}
```

**Data:**

```json
{
  "input": "Natural language processing is the field of study that focuses on the interactions between computers and human language."
}

{
  "object": "list",
  "data": [
    {
      "index": 0,
      "embedding": [0.123, -0.456, 0.789]
    }
  ],
  "usage": {
    "prompt_tokens": 15,
    "total_tokens": 15
  }
}

{
  "model": "text-davinci-003",
  "prompt": "Translate the following sentence to Spanish: 'The quick brown fox jumps over the lazy dog.'"
}

{
  "id": "cmpl-FooBar123",
  "object": "text_completion",
  "created": 1672598078,
  "model": "text-davinci-003",
  "choices": [
    {
      "index": 0,
      "text": "El rápido zorro marrón salta sobre el perro perezoso.",
      "logprobs": null,
      "finish_reason": "length"
    }
  ]
}
```

**Flow:**

1. Prompt
2. Embeddings Request
3. Input?
4. List of Embeddings
5. Usage Data
6. Chat Completion Request
7. Input?
8. Human-like Text

