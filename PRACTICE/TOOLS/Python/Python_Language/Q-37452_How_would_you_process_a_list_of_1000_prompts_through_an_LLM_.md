---
type: "practice"
difficulty: "Intermediate"
---

# How would you process a list of 1000 prompts through an LLM in Python without hitting rate limits, at a beginner-friendly level?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming LLM Responses](<../../../../TOOLS/Python/Python_Language/Streaming_LLM_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When processing a large number of prompts through an LLM (Language Model) using Python, one must carefully manage the API calls to avoid exceeding rate limits and ensure the process is efficient. In this scenario, we'll discuss how to handle 1000 prompts without hitting rate limits, focusing on intermediate Python skills.

The key steps involve:

Understanding the API's rate limit.
Implementing a mechanism to batch or spread out calls over time.
Error handling and retry logic in case of failures.

#### Triage

The question is asking for a strategy to process 1000 prompts through an LLM without hitting rate limits, ensuring that the code handles potential errors gracefully. The interviewer might be probing your understanding of API rate limiting, asynchronous operations, and error handling in Python.

#### Solution
from First Principles
Step 1: Understand Rate Limits

First, identify the rate limit imposed by the LLM's API documentation. For example, let’s assume we have a rate limit of 60 requests per minute (this is just an example; actual limits can vary).

PYTHON
RATE_LIMIT_REQUESTS = 60
RATE_LIMIT_PERIOD = 60  # seconds
Step 2: Batch Processing

To avoid hitting the rate limit, batch the prompts into smaller chunks that fit within the rate limit. For instance, we could use a chunk size of 50 requests per minute.

PYTHON
BATCH_SIZE = RATE_LIMIT_REQUESTS * RATE_LIMIT_PERIOD / 60

This gives us a batch size of 10 prompts (for simplicity in this example).

Step 3: Asynchronous Processing

Use asynchronous programming to process the batches concurrently. This can significantly speed up the overall processing time.

PYTHON
import asyncio

async def process_prompt(prompt):
    # Simulate an API call with some delay and potential errors
    import aiohttp
    async with aiohttp.ClientSession() as session:
        try:
            async with session.post('https://api.example.com/llm', json={'prompt': prompt}) as response:
                if response.status == 200:
                    return await response.json()
                else:
                    raise Exception(f"API error: {response.status}")
        except Exception as e:
            print(f"Error processing prompt: {e}")

async def batch_process(prompts):
    tasks = [process_prompt(prompt) for prompt in prompts]
    results = await asyncio.gather(*tasks)
    return results
Step 4: Scheduling Batches

Create a loop that processes batches of prompts, ensuring you stay within the rate limit. After processing one batch, wait until the next interval to start the next batch.

PYTHON
async def process_prompts(prompts):
    for i in range(0, len(prompts), BATCH_SIZE):
        batch = prompts[i:i + BATCH_SIZE]
        results = await batch_process(batch)
        # Process or save the results as needed

# Example usage
prompts = ["Prompt 1", "Prompt 2", ..., "Prompt 1000"]  # Assume a list of 1000 prompts
asyncio.run(process_prompts(prompts))

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that we do not exceed the rate limit by batching requests appropriately. Asynchronous programming is key to handling this efficiently. Common mistakes include not understanding or misinterpreting the API's rate limits, failing to handle errors properly, and not using asynchronous operations effectively.

A follow-up question might be: "How would you implement rate limiting at the application level if the LLM service does not provide a mechanism for it?"

