---
type: "practice"
difficulty: "Intermediate"
---

# How would you add a content moderation or safety check to a FastAPI LLM endpoint before returning generated text?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Batching LLM Requests](<../../../../TOOLS/Python/FastAPI/Batching_LLM_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To add a content moderation or safety check to a FastAPI LLM (Language Model) endpoint, we need to ensure that the generated text does not contain any prohibited content such as hate speech, explicit language, etc., before it is returned to the user. This involves several steps, including defining what constitutes unsafe content, implementing a filtering mechanism, and testing the solution thoroughly.

Firstly, let’s break down the requirements:

Define the criteria for safe vs unsafe text.
Integrate these checks into the FastAPI endpoint.
Ensure that the checks do not significantly impact performance or user experience.

Now, let's go through each step in detail:

Define Unsafe Content Criteria: We need to define a set of rules or patterns that would identify unsafe content. For simplicity, we can use regular expressions (regex) to match common patterns such as profanity, hate speech, and explicit language. These criteria will be stored in a configuration file.

Set Up FastAPI Endpoint: We’ll create an endpoint using FastAPI that accepts user input, generates text from the LLM, applies the safety checks, and returns the result or an error message if the content is unsafe.

Implement Safety Checks: We will use regex to check the generated text against our predefined criteria.

Testing and Validation: Finally, we need to test the endpoint with various inputs to ensure that it correctly identifies and blocks inappropriate content without affecting performance.

Let’s dive into each step in more detail:

1. Define Unsafe Content Criteria

We can define a simple set of regex patterns for unsafe content. For instance:

PYTHON
UNSAFE_PATTERNS = [
    r"\b(?:cunt|faggot)\b",  # Profanity words
    r"sex\s+with\s+a\s+minor",  # Explicit content
]
2. Set Up FastAPI Endpoint

We’ll create a FastAPI endpoint that receives text input from the user, generates text using an LLM like Anthropic's Claude or Cohere, and then applies our safety checks.

PYTHON
from fastapi import FastAPI, HTTPException
import re

app = FastAPI()

# Load unsafe patterns
UNSAFE_PATTERNS = [
    r"\b(?:cunt|faggot)\b",  # Profanity words
    r"sex\s+with\s+a\s+minor",  # Explicit content
]

@app.post("/generate_text")
async def generate_safe_text(input_text: str):
    import requests

    # Generate text using LLM API (example with Anthropic)
    response = requests.post("https://api.anthropic.com/v1/complete",
                             headers={"Authorization": "Bearer YOUR_API_KEY"},
                             json={"model": "claude-2", "prompt": input_text})

    generated_text = response.json()["completion"]

    # Apply safety checks
    for pattern in UNSAFE_PATTERNS:
        if re.search(pattern, generated_text):
            raise HTTPException(status_code=403, detail="Generated text contains unsafe content.")

    return {"text": generated_text}
3. Implement Safety Checks

In the example above, we are using re.search to check for any of our predefined patterns in the generated text. If a match is found, an HTTPException with status code 403 (Forbidden) is raised.

4. Testing and Validation

We should test the endpoint with various inputs, including both safe and unsafe content, to ensure that it works correctly:

Safe input: "I love programming."
Unsafe input: "Have you ever had sex with a minor?"

By running these tests, we can verify that our safety checks are effective.

#### Closing Summary

In summary, adding content moderation or safety checks to a FastAPI LLM endpoint involves defining unsafe content criteria using regex, implementing the checks in your API, and testing thoroughly. This ensures that the generated text is safe for public consumption while maintaining performance.

#### Common mistakes
include not covering all edge cases (e.g., variations of profanity), failing to test with realistic data, or not accounting for potential performance impacts due to frequent regex searches.

