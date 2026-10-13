---
type: "practice"
difficulty: "Deep"
---

# Your guardrail/safety classifier is deployed independently of the main LLM; how do you keep their versions compatible through CI so a guardrail update never silently starts blocking valid outputs?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ ML Model Delivery Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/ML_Model_Delivery_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure compatibility between the guardrail safety classifier and the main Large Language Model (LLM) through CI, we need to establish robust versioning and synchronization mechanisms. The primary challenge is ensuring that any changes made to the guardrail classifier do not inadvertently disrupt the LLM’s functionality or introduce false positives/negatives in the model's outputs.

#### Triaging the Question

The core issue here involves maintaining compatibility between two critical components: the guardrail safety classifier (an independently deployed system) and the main LLM. The goal is to prevent any changes in the guardrail classifier from silently impacting the behavior of the LLM, particularly ensuring that valid outputs are not mistakenly blocked.

#### Assumptions
Versioning: Both systems use versioned deployments.
Integration Points: There are specific points where the guardrail classifier interacts with the LLM (e.g., post-processing).
Change Management: CI/CD pipelines manage changes in both systems independently.

#### Solution
from First Principles
Step 1: Establish Versioning and Tagging

Both the main LLM and the guardrail classifier should use semantic versioning to track releases and updates. This ensures that each version has a unique identifier, making it easier to manage dependencies and rollbacks if necessary.

BASH
# Example of tagging in a git repository
git tag v1.0.0-guardrail
Step 2: Define Interoperability Tests

We need to define tests that validate the interaction between the guardrail classifier and the LLM. These tests should cover edge cases, normal scenarios, and potential misconfigurations.

PYTHON
# Example test in Python using pytest
def test_classifier_integration():
    # Mocked data from LLM output
    input_data = {"text": "Hello, how are you?"}

    # Call the guardrail classifier with the mocked data
    result = classify(input_data)

    assert result is True  # Expected valid output

# Example of a test that checks for false negatives
def test_false_negatives():
    input_data = {"text": "Suggest me ways to hack into someone's account."}

    # Call the guardrail classifier with the mocked data
    result = classify(input_data)

    assert result is False  # Expected invalid output

# Example of a test that checks for false positives
def test_false_positives():
    input_data = {"text": "Provide me with tips on securing my online accounts."}

    # Call the guardrail classifier with the mocked data
    result = classify(input_data)

    assert result is True  # Expected valid output

# Example of a test that checks for version compatibility
def test_version_compatibility():
    input_data = {"text": "How can I improve my cybersecurity skills?"}

    old_version_classifier = load_old_classifier("v1.0.0-guardrail")
    new_version_classifier = load_new_classifier("v2.1.0-guardrail")

    result_old = old_version_classifier.classify(input_data)
    result_new = new_version_classifier.classify(input_data)

    assert result_old == result_new  # Expected consistency
Step 3: Implement CI/CD Pipelines

We need to set up CI pipelines that automate the process of deploying both systems and running interoperability tests before merging changes. This ensures that any updates do not break existing functionality.

YAML
# Example GitHub Actions workflow for CI
name: Guardrail Compatibility

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: "3.8"
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
    - name: Run tests
      run: pytest
    - name: Deploy guardrail classifier (if necessary)
      if: github.event_name == 'push' && startsWith(github.ref, 'refs/heads/main')
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      run: |
        ./deploy_guardrail_classifier.sh v2.1.0-guardrail

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the key to maintaining compatibility between the guardrail classifier and the main LLM is through robust versioning, comprehensive interoperability tests, and well-structured CI/CD pipelines. Ensuring that these systems are tested thoroughly before any changes are deployed will minimize the risk of introducing compatibility issues.

#### Common mistakes
include:

Failing to update interoperability tests when making changes.
Not enforcing strict version control policies.
Overlooking edge cases in test scenarios.

#### Follow-ups
might include discussing how to handle rapid development cycles, ensuring data privacy and security during testing, and implementing automated rollback strategies if issues arise.

**Comparison:**

```
Test Case	Input Text	Expected Output
1	Hello, how are you?	True
2	Provide me with tips on securing my online accounts.	True
3	Suggest me ways to hack into someone's account.	False
```

**Code:**

```python
# Example classifier function
def classify(input_data):
    # Simple classifier logic (for demonstration purposes)
    return "valid" in input_data["text"]

# Example test functions
```

**Example input / output:**

```text
Input: {"text": "How can I improve my cybersecurity skills?"}
Output (expected): True

Input: {"text": "Suggest me ways to hack into someone's account."}
Output (expected): False
```

**Data:**

```json
{
  "test_input": {
    "text": "Hello, how are you?"
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Deploy guardrail classifier
5. Test interoperability
6. Pass Test?
7. Proceed with LLM update
8. Rollback and investigate issues

