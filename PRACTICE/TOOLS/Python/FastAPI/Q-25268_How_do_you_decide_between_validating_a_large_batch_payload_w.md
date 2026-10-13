---
type: "practice"
difficulty: "Deep"
---

# How do you decide between validating a large batch payload with a single Pydantic model containing a list versus streaming and validating items incrementally in FastAPI, and what are the memory trade-offs?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pydantic Validation](<../../../../TOOLS/Python/FastAPI/Pydantic_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether to validate a large batch payload using a single Pydantic model containing a list or to stream and validate items incrementally in FastAPI, you need to consider several factors, including performance, memory usage, and error handling. Let’s break down the core considerations step by step.

Firstly, let's understand what we are validating:

Single Pydantic Model: You define a Pydantic model that represents your payload as a list of items.
Incremental Validation: You validate each item individually as it comes in.

#### Triage
What is the Question Really Asking?

The question is probing how to balance validation efficiency and memory usage, especially for large payloads. We need to consider:

Memory Usage: How much memory does each approach consume?
Validation Efficiency: Is one method faster than the other?
Error Handling: How do errors affect each validation strategy?

#### Assumptions
The payload is a list of JSON objects, where each object needs to be validated.
FastAPI’s Pydantic models are used for validation.

#### First-Principles Solution
Single Pydantic Model (Batch Validation)
Memory Usage: When using a single model with a List field, the entire payload is loaded into memory at once. This can lead to high memory usage if the batch size is large.
Validation Efficiency: The validation process is typically faster because it happens in one go after all data has been received and parsed.
Error Handling: If any item fails validation, you need to parse the entire payload again or handle the error in a post-processing step.
PYTHON
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    id: int
    name: str
    value: float

app = FastAPI()

@app.post("/items")
async def create_items(items: List[Item]):
    # Validation happens here after all items are received
Incremental Validation (Streaming)
Memory Usage: Each item is validated as it comes in, so memory usage is proportional to the number of currently validated items.
Validation Efficiency: Potentially slower due to repeated validation and parsing steps for each item.
Error Handling: Errors can be handled immediately upon discovery, reducing the need for reprocessing.
PYTHON
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.post("/items")
async def create_items(item: Item):
    # Validation happens here on a per-item basis
    if not valid_item(item):
        raise HTTPException(status_code=422, detail="Item is invalid")

def valid_item(item: Item):
    # Custom validation logic
    pass

#### Closing Summary
and Common Mistakes/Follow-Ups
Memory Trade-offs: Batch validation uses more memory but can be faster if the payload size is large.
Error Handling: Incremental validation offers better error handling, especially for large payloads where partial success might still be useful.

#### Implementation
Complexity: Incremental validation may require additional logic to manage state and errors during processing.

#### Common mistakes
include:

Overlooking the memory impact of batch validation on very large datasets.
Ignoring the potential overhead of incremental validation, which can add complexity and reduce overall performance in some cases.

For deeper analysis, you might also consider how network bandwidth and latency affect streaming versus batch approaches, as well as the specific constraints of your application.

**Comparison:**

```
Validation Approach	Memory Usage	Validation Efficiency	Error Handling
Batch (Single Model)	High	Fast	Reprocess or Post-process Errors
Incremental	Low	Variable	Immediate
```

**Example input / output:**

```text
Input: [ {"id": 1, "name": "Item A", "value": 10}, {"id": 2, "name": "Item B", "value": -5}, {"id": 3, "name": "Item C", "value": 7} ]

Output:
- Validated Item A
- Invalidated Item B (negative value for 'value')
- Validated Item C

Error Response:
HTTP 422 Unprocessable Entity with detail: "Item is invalid"
```

**Data:**

```json
[
    {"id": 1, "name": "Item A", "value": 10},
    {"id": 2, "name": "Item B", "value": -5},
    {"id": 3, "name": "Item C", "value": 7}
]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Receive Payload
6. Is Single Pydantic Model?
7. Validate All Items Once
8. Possibly High Memory Usage
9. Process Item by Item
10. Validate Each Item Individually
11. Handle Error Immediately?
12. Return Error Response
13. Continue ProcessingRemaining Items

