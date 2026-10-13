---
type: "practice"
difficulty: "Intermediate"
---

# When you enable streaming on an OpenAI request, what field do you set in the request body, and how does the response format change?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming API Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Streaming_API_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer your question about enabling streaming on an OpenAI request, we first need to understand that streaming involves sending responses from the API as they become available rather than waiting until all data is ready. In terms of the request and response body, the key change lies in setting a specific parameter in the request.

TL;DR

When you enable streaming, you set the stream parameter to true or include &stream=true in your API URL query string for most endpoints. This enables the server to send responses incrementally as they are generated, rather than buffering and sending them all at once. The response format changes accordingly to deliver data in chunks.

#### Triage

The question is essentially asking about how enabling streaming affects both the request and the response formats when using OpenAI's API. We need to identify the parameter that triggers this behavior and understand its implications on the response structure.

#### First-Principles Solution
Request Body Changes

For most endpoints, including streaming requires modifying the stream parameter in the request URL rather than the body. This is because many APIs, including OpenAI's, use query parameters for controlling streaming behavior. Here’s an example of setting up a streaming request:

HTTP
GET https://api.openai.com/v1/engines/davinci/completions?stream=true&prompt=Write%20a%20short%20story%20about%20a%20dog.

In this example, stream=true is added as a query parameter to the URL. The API will respond with data in chunks rather than waiting until it has generated the full response.

Response Format Changes

When streaming is enabled, the server sends responses in smaller chunks instead of buffering and sending them all at once. This means that your client needs to handle these incremental updates properly by processing each chunk as they arrive. The exact format of these chunks can vary depending on the API's implementation, but generally, you might see JSON objects or plain text.

#### Example
Response Format

Here’s an example of how a streamed response might look:

JSON
{
  "id": "cmpl-6G1234",
  "object": "text_completion",
  "created": 1628940784,
  "model": "text-davinci-002",
  "choices": [
    {
      "index": 0,
      "text": ".\nOnce upon a time, in a small village...",
      "logprobs": null,
      "finish_reason": "length"
    }
  ]
}

This chunk might be followed by more chunks as the API generates additional content. The client should handle each choices object separately and concatenate them until the stream is complete.

#### Closing Summary

Enabling streaming on an OpenAI request involves setting a specific parameter (stream=true) in the query string rather than the body. This triggers the server to send responses incrementally, which requires your client to process these chunks of data as they arrive. The response format changes to deliver content in smaller, manageable parts.

#### Common Mistakes
and Follow-Ups
Incorrect Parameter: Sometimes developers might mistakenly place stream=true in the request body instead of the query string.
Handling Chunks: Clients need to be prepared to process these incremental responses efficiently without waiting for the entire response to arrive.
Timeouts and Errors: Streaming can introduce challenges with timeouts and error handling, as partial results may be received before the connection is closed.

By understanding these nuances, you can ensure that your API interactions are robust and efficient when dealing with large or time-consuming operations.

