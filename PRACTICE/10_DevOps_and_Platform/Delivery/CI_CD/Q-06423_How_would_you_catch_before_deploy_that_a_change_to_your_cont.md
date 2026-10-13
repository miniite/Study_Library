---
type: "practice"
difficulty: "Deep"
---

# How would you catch, before deploy, that a change to your context-truncation logic causes the model to drop the retrieved evidence and hallucinate more?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Infrastructure Drift Detection](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Infrastructure_Drift_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure we don't deploy changes that break our model's ability to retrieve relevant evidence without hallucinating, we need a robust CI/CD pipeline with comprehensive automated tests. The key is to catch such issues early in the development process by implementing end-to-end testing and validation steps focused on context-truncation logic.

The question here involves understanding how a change in context truncation can affect the model's ability to retrieve accurate evidence, which could lead to hallucinations if not properly validated. We need to design tests that check both the retrieval of relevant evidence and the absence of irrelevant information introduced as hallucinations.

**Comparison:**

| Scenario | Context Length (Chars) | Expected Evidence Snippets       |
|----------|-----------------------|---------------------------------|
| Normal   | 200                   | ["Feature A", "Feature B"]       |
| Long     | 150                   | []                              |
| Short    | 300                   | ["Feature A, Feature B"]         |

**Code:**

```python
# Example setup for a test in a CI/CD pipeline

def test_context_truncation():
    # Mock input data
    input_text = "User query about product features."

    # Define expected context length and evidence snippets
    expected_context_length = 200
    expected_evidence_snippets = ["Feature A: Detailed description", "Feature B: Another detailed description"]

    # Simulate model response
    model_response = {"context": input_text[:expected_context_length],
                      "evidence": [snippet for snippet in expected_evidence_snippets if len(snippet) <= 100]}

    # Validate context length and evidence snippets
    assert len(model_response["context"]) == expected_context_length, "Context is truncated incorrectly."
    assert set(model_response["evidence"]) == set(expected_evidence_snippets), "Incorrect evidence snippets returned."

# Run the test in CI/CD pipeline
test_context_truncation()
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "input_text": "User query about product features.",
  "expected_context_length": 200,
  "expected_evidence_snippets": ["Feature A: Detailed description", "Feature B: Another detailed description"]
}

Output:

PYTHON
Copy
True # Test passes if the context and evidence snippets match expectations.
```

**Data:**

```json
{
  "input_text": "User query about product features.",
  "expected_context_length": 200,
  "expected_evidence_snippets": ["Feature A: Detailed description", "Feature B: Another detailed description"]
}
```

**Flow:**

1. Start CI/CD Pipeline
2. Run Unit Tests
3. Check Context Length andEvidence Snippets
4. Pass or Fail
5. Report Results, Alert Team
6. Manual Review if Necessary
7. Deploy if No Issues

