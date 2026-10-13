---
type: "practice"
difficulty: "Basic"
---

# How would you count or estimate the number of tokens in a prompt before sending it to an LLM from a FastAPI endpoint?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ LLM Endpoints in FastAPI](<../../../../TOOLS/Python/FastAPI/LLM_Endpoints_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To count or estimate the number of tokens in a prompt sent via a FastAPI endpoint, we need to understand that each character in the input string is typically tokenized by the tokenizer used with your language model (LLM). For instance, if you're using a transformer-based LLM like those from Hugging Face’s transformers library, the tokenizer will split the prompt into tokens. We can leverage this by sending the prompt through the tokenizer and counting the resulting tokens.

#### Triage

The question is essentially asking for a method to count or estimate the token length of a prompt before it gets sent to an LLM via a FastAPI API endpoint. This involves understanding how tokenization works, knowing which tokenizer to use, and implementing a quick-and-dirty or precise method based on your familiarity with the system.

#### First-Principles Solution
Identify the Tokenizer: Determine which tokenizer is being used by your LLM. Common choices include Hugging Face's AutoTokenizer.
Tokenize the Prompt: Use this tokenizer to convert the prompt into tokens.
Count Tokens: The length of the resulting token list will give you the number of tokens.
Code Implementation

Here’s a basic implementation in Python using FastAPI and Hugging Face’s transformers library:

PYTHON
from fastapi import FastAPI, Body
import torch
from transformers import AutoTokenizer

app = FastAPI()

# Load your tokenizer (replace 'bert-base-uncased' with your actual model)
tokenizer = AutoTokenizer.from_pretrained('bert-base-uncased')

@app.post("/count_tokens/")
async def count_tokens(prompt: str):
    # Tokenize the prompt and get the tokens
    encoded_input = tokenizer(prompt, return_tensors='pt')

    # The number of tokens is the length of the input_ids tensor
    num_tokens = len(encoded_input['input_ids'][0])

    return {"num_tokens": num_tokens}

#### Closing Summary
+ Common Mistakes/Follow-Ups

In this solution, we’ve defined a FastAPI endpoint that receives a prompt and returns its token count. The key steps are identifying the correct tokenizer and using it to tokenize the input string.

#### Common Mistakes

Not properly importing necessary libraries.
Using the wrong tokenizer or model name.
Not converting the tensor back to a list before counting.

#### Follow-Ups

What if we need to handle multiple prompts at once?
How can we optimize this process for larger inputs?

