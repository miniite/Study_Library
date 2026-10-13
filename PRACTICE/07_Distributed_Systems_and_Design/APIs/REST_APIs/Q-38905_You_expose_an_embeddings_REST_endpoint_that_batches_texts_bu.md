---
type: "practice"
difficulty: "Intermediate"
---

# You expose an /embeddings REST endpoint that batches texts, but one oversized text in a batch fails the whole call. How do you design partial-success semantics so valid texts return vectors and only the bad ones are reported?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Vector Search APIs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Vector_Search_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing an /embeddings REST endpoint to handle batches of texts, one common issue is handling oversized texts that can fail the entire batch call. To ensure partial success—where valid texts return their vectors while the bad ones are reported—the design needs to be robust and flexible. Here’s a step-by-step approach:

TL;DR

To achieve partial-success semantics in an /embeddings REST endpoint, we need to process each text individually, handle exceptions for oversized texts, and provide clear feedback on which texts failed and why.

#### Triaging the Question

The core of this question revolves around designing robust error handling that can distinguish between valid inputs and problematic ones. The key is ensuring that:

Partial Success: Valid texts should return their embeddings.
Error Reporting: Oversized or otherwise invalid texts should be reported with specific errors.

#### Assumptions
and Reasoning

Assume the API receives a JSON payload containing an array of text strings. Each text string is processed individually to determine if it can be embedded. If any single text exceeds a predefined size limit, that particular text should not prevent others from being processed.

Step 1: Define Input Schema

The input will be a JSON object with an array of texts.

JSON
{
    "texts": ["Text 1", "Text 2", ..., "Too Large Text"]
}
Step 2: Size Validation Logic

Each text string needs to be validated against a maximum size limit. If any text exceeds this limit, it should trigger an error.

Step 3: Process Each Text Individually

Process each text in the array and handle exceptions for oversized texts. Store results and errors separately.

PYTHON
def embed_texts(texts, max_size):
    embeddings = []
    errors = []

    for text in texts:
        try:
            # Assume this function processes a single text and returns its embedding
            embedding = process_text(text)
            embeddings.append(embedding)
        except TextTooLargeError as e:
            errors.append((text, str(e)))

    return {"embeddings": embeddings, "errors": errors}

#### First-Principles Solution
Input Validation: Validate the input to ensure it is a JSON array of texts.
Individual Processing: Process each text individually and catch any exceptions that arise from size validation or processing.
Error Handling and Reporting: Maintain separate lists for valid embeddings and errors, ensuring clear feedback on which texts failed.

#### Closing Summary

By following these steps, we can design an /embeddings endpoint that processes batches of texts with partial success semantics. This ensures that even if one oversized text causes an issue, the rest of the batch continues to process successfully, providing both a useful result and clear error messages for problematic inputs.

#### Common Mistakes/Follow-Ups
Ignoring Oversized Texts: Failing to handle oversized texts can lead to partial success issues.
Inconsistent Error Reporting: Not clearly reporting which texts failed and why could make debugging difficult.
Overly Complicated Exception Handling: Over-engineering the exception handling can introduce unnecessary complexity.
Additional Considerations
Throttling Mechanism: Implement rate limiting or throttling mechanisms to prevent abuse from clients sending excessively large batches.
Logging and Monitoring: Log errors for further analysis and monitoring to understand failure patterns better.

**Comparison:**

```
Text	Embedding	Error Message
Text 1	[hash('Text 1')]	-
Short text	[hash('Short text')]	-
This text is too large and will cause an error.	-	Text exceeds maximum allowed length
```

**Code:**

```python
class TextTooLargeError(Exception):
    pass

def process_text(text, max_size=1024):  # Example size limit
    if len(text) > max_size:
        raise TextTooLargeError("Text exceeds maximum allowed length")

    # Simulate embedding processing
    return [hash(text)]  # Placeholder for actual embedding logic

def embed_texts(texts):
    embeddings = []
    errors = []

    for text in texts:
        try:
            embedding = process_text(text)
            embeddings.append(embedding)
        except TextTooLargeError as e:
            errors.append((text, str(e)))

    return {"embeddings": embeddings, "errors": errors}
```

**Example input / output:**

```text
result = embed_texts(["Text 1", "This text is too large and will cause an error.", "Short text"])
print(result)
# Output:
{
    "embeddings": [[[hash('Text 1')]], [[hash('Short text')]]],
    "errors": [
        ("This text is too large and will cause an error.", "Text exceeds maximum allowed length")
    ]
}
```

**Data:**

```json
{
    "texts": ["Text 1", "This text is too large and will cause an error.", "Short text"]
}
```

**Flow:**

1. Valid JSON array
2. Receive Batch of Texts
3. Validate Input
4. Process Each Text
5. Process
6. [Text 1], [Text 2], ..., [Too Large]
7. Failed Validation
8. Store Error
9. Return Results

