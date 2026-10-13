---
type: "practice"
difficulty: "Basic"
---

# How would you use Copilot to add a maximum output length (max tokens) to an LLM call, and why does it matter for cost and latency?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Verifying AI Generated ML Code](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Verifying_AI_Generated_ML_Code.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To add a maximum output length (max_tokens) constraint when using Copilot, you would typically set this parameter in the API call to the language model. This is crucial because setting max_tokens can significantly reduce both cost and latency by limiting the amount of computation required.

#### Triaging the Question
The question revolves around understanding how to optimize an LLM (Language Model) call using Copilot, specifically focusing on the max_tokens parameter. The interviewer is probing your knowledge of API parameters, their importance in controlling model behavior, and how these parameters impact cost and latency.

Key Assumptions:

You are familiar with basic concepts of AI-assisted coding.
You understand that LLMs like Copilot make API calls to generate text based on given prompts.
You recognize the importance of performance optimization (cost and latency) in real-world applications.

Step-by-Step Solution:

Understanding the Parameter: The max_tokens parameter sets the maximum number of tokens (words or subwords) that Copilot can return as output. In most API calls, this is a required or optional field.

API Call Example: When making an API call to an LLM, you might use something like:

PYTHON
response = copilot.generate(prompt=prompt, max_tokens=100)

Impact on Cost and Latency:

Cost: Reducing max_tokens directly decreases the computational resources required by the model to generate text. Models generally consume more tokens, meaning they incur higher costs.
Latency: A lower max_tokens value reduces the time it takes for the API call to complete, as less computation is involved.

#### Example Scenario

Suppose you are working on an application that frequently generates short summaries (e.g., 50 tokens). Setting max_tokens=50 ensures efficient processing without generating unnecessary text.

#### Closing Summary
and Common Mistakes/Follow-Ups: In summary, setting the max_tokens parameter in Copilot or any LLM call is essential for balancing quality of output with cost-efficiency and performance. Overusing tokens can lead to higher costs and longer response times, which are critical considerations in practical applications.

#### Common Mistakes

Failing to set a reasonable limit on max_tokens.
Ignoring the impact of token usage on both cost and latency.

#### Follow-ups
might include discussing trade-offs between model accuracy and resource consumption, or exploring other performance optimization techniques like caching intermediate results or batching API calls.

**Code:**

```python
response = copilot.generate(prompt=prompt, max_tokens=100)
```

