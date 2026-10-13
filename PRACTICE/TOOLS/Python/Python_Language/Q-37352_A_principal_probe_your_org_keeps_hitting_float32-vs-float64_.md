---
type: "practice"
difficulty: "Deep"
---

# A principal probe: your org keeps hitting float32-vs-float64 drift between pandas preprocessing and the trained model. How would you enforce a consistent-precision contract across the whole Python scoring path platform-wide, and detect violations in CI?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Drift-triggered Retraining](<../../../../TOOLS/Python/Python_Language/Python_Tail_Latency/Drift-triggered_Retraining.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure consistency in floating-point precision across your organization's entire Python scoring pipeline—from data preprocessing to model deployment—ensuring a consistent-precision contract is crucial. This involves defining and enforcing strict rules at every step of the pipeline, particularly where pandas and machine learning models interact. The goal is not only to maintain precision but also to detect any potential drift during Continuous Integration (CI) tests.

The core challenge here revolves around the differences between float32 and float64, which can lead to subtle but significant discrepancies in model predictions. By enforcing a consistent data type, we can mitigate these issues and ensure reliable results across all components of our system.

#### Triage
What's Being Probed

The question probes your understanding of:

Precision Management: How different floating-point types impact numerical computations.
Data Pipeline Consistency: Ensuring that the same data types are used consistently throughout a pipeline.
CI/CD Integration: Automating checks to detect and prevent precision-related issues.

#### Assumptions
The organization uses both float32 (common in training) and float64 (used in some models for higher precision).
There is a need to maintain consistency across the entire pipeline, from data preprocessing through model inference.
CI systems are already in place but may lack specific checks related to floating-point precision.

#### Solution
Enforcing Consistency and Detecting Violations

Define Precision Contracts: Establish clear rules for when float32 and float64 should be used. For example, float64 might be required only in specific parts of the pipeline where precision is critical.

Data Type Enforcement: Implement checks at every stage to ensure that data types align with defined contracts. This can be achieved by using custom validation functions or leveraging libraries like numpy.

#### Cons
istency Checks in CI/CD:

Preprocessing Stage: Ensure that all input data is converted to the appropriate type before it reaches downstream processes.
Model Inference: Verify that model inputs and outputs match expected types.

Automated Testing Frameworks: Use unit tests or custom scripts to run these checks automatically in CI pipelines.

Logging and Monitoring: Integrate logging mechanisms to capture any data type mismatches for easier debugging and root cause analysis.

Detailed Solution
Step 1: Define Precision Contracts

First, define the precision requirements for different stages of your pipeline. For instance:

PYTHON
class PrecisionConfig:
    PREPROCESSING = "float32"
    INFERENCE_INPUT = "float64"
    INFERENCE_OUTPUT = "float64"

These constants ensure that all components follow a predefined standard.

Step 2: Data Type Enforcement

Implement functions to enforce these data types. For example, a function might look like:

PYTHON
def enforce_precision(data, expected_type):
    if isinstance(data, np.ndarray) and not data.dtype == expected_type:
        return data.astype(expected_type)
    return data

# Example usage in preprocessing stage
preprocessed_data = enforce_precision(preprocessed_data, np.float32)
Step 3: Consistency Checks in CI/CD

Add checks to your CI pipelines using tools like pytest or custom scripts. For instance:

PYTHON
def test_data_type_consistency(data):
    assert data.dtype == expected_dtype

# Example check for preprocessing stage
test_data_type_consistency(preprocessed_data)
Step 4: Automated Testing Frameworks

Integrate these checks into your CI/CD pipeline. This could be done using Jenkins, GitLab CI, or any other CI tool.

YAML
# Example GitLab CI YAML snippet
stages:
  - preprocess
  - test

preprocess_job:
  stage: preprocess
  script:
    - python3 preprocess.py

test_job:
  stage: test
  script:
    - pytest --collect-only
Step 5: Logging and Monitoring

Implement logging to capture any data type mismatches:

PYTHON
import logging

logger = logging.getLogger(__name__)

def enforce_precision(data, expected_type):
    if isinstance(data, np.ndarray) and not data.dtype == expected_type:
        logger.error(f"Data type mismatch: {data.dtype} != {expected_type}")
        return data.astype(expected_type)
    return data

# Example logging in preprocessing
preprocessed_data = enforce_precision(preprocessed_data, np.float32)

#### Closing Summary
+ Common Mistakes/Follow-Ups

By enforcing a consistent-precision contract and integrating these checks into your CI/CD pipeline, you can ensure that floating-point precision issues are caught early. Key mistakes to avoid include ignoring edge cases where data types might be implicitly changed, and not having robust logging mechanisms for debugging.

Common Follow-Ups
How would you handle situations where precision is critical but computational resources are limited?
Can you discuss the trade-offs between using float32 vs float64 in different parts of the pipeline?

**Comparison:**

```
Preprocess Stage
Enforce Precision
Test Job
Test Data Type Consistency
```

**Code:**

```python
class PrecisionConfig:
    PREPROCESSING = "float32"
    INFERENCE_INPUT = "float64"
    INFERENCE_OUTPUT = "float64"

def enforce_precision(data, expected_type):
    if isinstance(data, np.ndarray) and not data.dtype == expected_type:
        return data.astype(expected_type)
    return data
```

**Example input / output:**

| Input Data Type | Expected Output Type |
|-----------------|---------------------|
| float32         | float64             |
| float64         | float64             |

**Data:**

```yaml
stages:
  - preprocess
  - test

preprocess_job:
  stage: preprocess
  script:
    - python3 preprocess.py

test_job:
  stage: test
  script:
    - pytest --collect-only
```

**Flow:**

1. preprocess_job --> enforce_precision -> test_job
2. test_job --> test_data_type_consistency

