---
type: "practice"
difficulty: "Basic"
---

# How would you use curl to test an LLM chat completion API from the command line, including the key and JSON body?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming API Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Streaming_API_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To test an LLM chat completion API using curl from the command line, we need to send a POST request with the appropriate headers and JSON payload that includes our API key and the message or prompt we want the model to respond to. Here’s how you can do it:

Understand the Question: The question is asking for a method to use curl to interact with an LLM (Language Model) chat completion API endpoint, ensuring security by including an API key in the headers and providing necessary parameters in JSON format.

Key Points:

The API endpoint URL needs to be known.
Use POST request since we are sending data to the server.
Include a header with the API key for authentication.
Send a JSON body containing the prompt or message you want the model to process.

Step-by-Step Solution:

SH
# Example command using curl
curl --location --request POST 'https://api.example.com/v1/completions' \
     --header 'Authorization: Bearer YOUR_API_KEY' \
     --header 'Content-Type: application/json' \
     --data-raw '{
       "prompt": "What is the capital of France?",
       "max_tokens": 20
     }'

#### Closing Summary
and Common Mistakes:

Ensure you replace YOUR_API_KEY with your actual API key.
Check that the URL, headers, and JSON body match what your LLM provider expects.
Make sure to handle any potential errors or unexpected responses.

**Code:**

```
SH
Copy
# Example command using curl
curl --location --request POST 'https://api.example.com/v1/completions' \
     --header 'Authorization: Bearer YOUR_API_KEY' \
     --header 'Content-Type: application/json' \
     --data-raw '{
       "prompt": "What is the capital of France?",
       "max_tokens": 20
     }'
```

