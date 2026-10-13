---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design a Python test harness that detects prompt/model regressions with golden-file snapshots and semantic (not exact-string) assertions, and how you keep it from being either flaky or too permissive.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Offline Evaluation Harnesses](<../../../../TOOLS/Python/Python_Language/Safe_User-authored_Configuration/Offline_Evaluation_Harnesses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust Python test harness that detects prompt/model regressions using golden file snapshots and semantic assertions, we need to ensure both reliability and precision in our tests. This involves creating a system that can compare model outputs against expected results (golden files) while allowing for some flexibility due to the nature of natural language processing (NLP). Here’s a step-by-step breakdown:

#### Overview
The core idea is to leverage golden file snapshots as a baseline and use semantic analysis tools like difflib in Python to compare model outputs with expected results. By implementing both exact string matches and more lenient semantic checks, we can ensure the test harness is robust yet flexible enough to handle minor variations due to factors like data preprocessing or model changes.

#### Triage
The key elements being probed are:

How you define and manage golden files.
Techniques for comparing outputs with a high degree of accuracy.
Strategies to prevent flakiness (false positives/negatives).
Methods to maintain the test harness’s relevance over time as models evolve.

#### First-Principles Solution

Golden Files: Create golden files that contain the expected output for various prompts. These can be stored in version control systems like Git.
Semantic Comparison: Use libraries such as difflib to perform a line-by-line comparison of outputs against the golden files, allowing for some tolerance in differences (e.g., minor changes in formatting).
Scoring Mechanism: Implement a scoring mechanism that assigns a score based on how closely the model’s output matches the expected result. This can involve calculating similarity metrics like Levenshtein distance or Jaccard index.
Flakiness Mitigation:
Use consistent test environments to ensure repeatable results.
Introduce randomization checks to verify that the model produces similar outputs under varying conditions.
Implement retries for transient errors and logging to track the reasons behind failed tests.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

The key is to balance between strict exact string matching, which can be brittle, and overly lenient semantic checks that might fail to detect significant issues.
A common mistake is relying solely on exact string matches, leading to false negatives due to minor discrepancies. Another issue could be setting the tolerance level too high, making the test too permissive.

**Comparison:**

```
Metric	Value
Similarity Threshold	0.75 - 0.95
Retries on Failure	3 times with a 1-minute delay each time
Randomization Tests	Yes, varying input data and model parameters
```

**Code:**

```python
import difflib

def generate_similarity_score(output, golden_file):
    with open(golden_file) as f:
        expected_output = f.read()

    # Normalize both outputs to ignore formatting differences
    normalized_output = normalize_text(output)
    normalized_expected_output = normalize_text(expected_output)

    # Calculate similarity score using difflib
    seq_matcher = difflib.SequenceMatcher(None, normalized_output, normalized_expected_output)
    similarity_score = seq_matcher.ratio()

    return similarity_score

def normalize_text(text):
    # Implement text normalization logic here (e.g., lowercasing, removing punctuation)
    return text.lower().replace(',', '').replace('.', '')

# Example usage
output_file = "model_output.txt"
golden_file = "golden_output.txt"

score = generate_similarity_score(output_file, golden_file)
print(f"Similarity Score: {score}")
```

**Example input / output:**

```text
Input:
- Model Output File: "path/to/model_output.txt"
- Golden Output File: "path/to/golden_output.txt"

Output:
Similarity Score: 0.85
```

**Data:**

```json
{
  "model_output": "This is a sample output from the model.",
  "golden_output": "this is an example of the expected output."
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Yes
6. No
7. Start
8. Is Model Output Stable?
9. Run Retries
10. Analyze Failure Log
11. Is Issue Resolved by Retries?
12. Mark as Passing
13. Fail Test and Notify Team
14. Generate Similarity Score
15. Score >= Threshold?
16. Pass Test
17. Notify Developer for ManualReview

