---
type: "practice"
difficulty: "Basic"
---

# What does the JSON request body of a chat completion call usually contain, and what are the roles like system, user, and assistant?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming API Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Streaming_API_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When making a chat completion call using a REST API, the JSON request body typically contains several fields that provide context to the model about the interaction. The core of this is often a messages array, which includes messages from different participants in the conversation—usually denoted as system, user, and assistant roles. Here’s a breakdown:

TL;DR: A chat completion call's request body usually consists of a JSON object with a messages array that lists messages attributed to specific roles like system, user, or assistant.

#### Triage
The question is asking about the structure of the JSON request body and specifically delves into the roles (system, user, and assistant). It aims to understand how these elements are used to provide context for the model.

#### First Principles Solution

Message Structure: Each message within the messages array typically includes a role field and a content field.
Role Definitions:
System: Typically represents instructions or initialization messages provided by the API caller to set context, such as defining a scenario.
User: Represents the user's input in the conversation.
Assistant: Represents the model’s response to the previous message.

#### Closing Summary
and Common Mistakes/Follow-Ups:

Common Structure Example:

JSON
{
  "model": "gpt-3.5-turbo",
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "What is the weather like today?"},
    {"role": "assistant", "content": "I don't have real-time data, but typically it's sunny around this time of year."}
  ]
}

#### Common Mistakes

Forgetting to include a model field or specifying the wrong model.
Not providing clear roles for each message in the messages array, leading to ambiguity.

**Code:**

```json
{
   "model": "gpt-3.5-turbo",
   "messages": [
     {"role": "system", "content": "You are a helpful assistant."},
     {"role": "user", "content": "What is the weather like today?"},
     {"role": "assistant", "content": "I don't have real-time data, but typically it's sunny around this time of year."}
   ]
 }
```

