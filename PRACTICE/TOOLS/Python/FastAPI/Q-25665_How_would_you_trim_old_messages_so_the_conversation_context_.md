---
type: "practice"
difficulty: "Intermediate"
---

# How would you trim old messages so the conversation context stays under the model's token limit in a FastAPI chat endpoint?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Caching Predictions](<../../../../TOOLS/Python/FastAPI/Caching_Predictions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle message trimming in a FastAPI chat endpoint, our goal is to ensure that the conversation context remains within the model's token limit while preserving recent or relevant messages. This involves removing older, less critical messages that exceed the allowed token count.

#### Triaging the Question

The core of this question is about implementing logic to maintain an appropriate length for the input context provided to a language model in a chat application. The key aspects are:

Understanding Token Limits: Familiarity with how tokens work and the maximum limit.
Context Preservation: Ensuring that important messages, especially recent ones, remain intact.

#### Implementation
Details: How to design the FastAPI endpoint to handle this efficiently.

#### Assumptions
We are working within a typical token limit of 1024 for a model like GPT-3 (this can vary).
Messages are stored in a list or similar data structure, and we need to remove older messages as necessary.
Each message has a timestamp, which allows us to prioritize based on recency.

#### Solution

To implement this, follow these steps:

Define the Token Limit: Understand the maximum number of tokens allowed by your model.
Fetch All Messages: Retrieve all existing messages from the database or storage.
Tokenize Messages: Count the total number of tokens in the current context.
Trim Excess Tokens: If the token count exceeds the limit, remove older messages until the count is within bounds.
Maintain Recent Context: Ensure that recent and important messages are not removed.

Here’s a step-by-step approach to implement this:

Import Necessary Libraries:

PYTHON
from fastapi import FastAPI, Depends
from sqlalchemy.orm import Session

app = FastAPI()

Define the Token Limit:

PYTHON
MAX_TOKENS = 1024  # Example token limit for GPT-3

Fetch and Tokenize Messages: Assume we have a database model Message with fields content and timestamp.

PYTHON
from sqlalchemy import select

async def get_all_messages(session: Session):
    result = await session.execute(select(Message))
    return [msg.content for msg in result.scalars().all()]

Count Tokens: Use a function to count the tokens.

PYTHON
def count_tokens(messages):
    # Placeholder function; use actual tokenization logic here.
    return sum(len(msg.split()) for msg in messages)

Trim Messages:

PYTHON
async def trim_messages(session: Session, max_tokens=MAX_TOKENS):
    messages = await get_all_messages(session)
    current_token_count = count_tokens(messages)

    while current_token_count > max_tokens and len(messages) > 0:
        # Remove the oldest message.
        messages.pop(0)
        current_token_count -= len(messages[0].split()) if messages else 0

    return messages

Endpoint Implementation: Create a FastAPI endpoint to handle this logic.

PYTHON
from fastapi.responses import JSONResponse

@app.post("/trim_messages")
async def trim_messages_endpoint(session: Session = Depends(get_session)):
    trimmed_messages = await trim_messages(session)
    return JSONResponse(content={"trimmed_messages": trimmed_messages})

#### Closing Summary
and Common Mistakes

To summarize, the key is to dynamically manage message length based on token limits while ensuring recent messages are preserved. Common mistakes include:

Ignoring tokenization logic when counting tokens.
Not considering edge cases where there aren’t enough messages.
Failing to handle session dependencies properly in a FastAPI context.

