---
type: "practice"
difficulty: "Deep"
---

# Your CI runs the full agent eval suite (tool-calling trajectories) on every PR and it costs $200/run in API calls; how would you cut that cost without losing confidence that agent behavior didn't regress?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ CI Cost Optimization](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_Cost_Optimization/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the high cost of running the full agent eval suite (tool-calling trajectories) on every PR, while maintaining confidence in the agent's behavior, we need to balance the trade-offs between thoroughness and efficiency. This involves implementing a stratified testing approach that reduces redundant evaluations without compromising the integrity of the test results.

#### Triaging the Question

The core issue here is cost optimization while ensuring the reliability of CI/CD pipelines. The question probes our ability to design efficient testing strategies, understand the costs associated with each evaluation step, and implement dynamic and adaptive testing techniques.

#### Assumptions
and Key Concepts
Cost: Each run of the full suite costs $200 in API calls.
Test Suite: The full eval suite is comprehensive but may contain redundant or less critical tests.
CI/CD Pipeline: Frequent PRs are evaluated, leading to high costs over time.

#### Solution
from First Principles
Dynamic Testing Stratification:
Implement a stratified testing approach where the system automatically determines which parts of the eval suite need full coverage and which can be sampled.
Threshold-Based Sampling:
Use statistical sampling techniques based on recent changes in PRs to decide how many random samples are needed from each test category.
Branch-Specific Testing:
Focus testing efforts only on branches that have shown instability or significant changes, reducing the overall number of full-suite runs.
Step-by-Step Solution
Analyze Recent Changes:
Use Git diff to identify which files or modules in a PR have changed.
Dynamic Test Selection:
Based on the analysis, select relevant test cases from the eval suite that are most likely affected by changes.
Statistical Sampling:
For unchanged parts of the codebase, randomly sample tests within each category to ensure coverage without full runs.
Thresholds and Feedback Loops:
Implement thresholds for when a high number of failures or unstable behavior requires a full suite run.
Adaptive Testing Framework:
Develop an adaptive framework that continuously learns from past test outcomes and PR history, optimizing the testing strategy over time.

#### Closing Summary

By implementing a stratified, dynamic testing approach, we can significantly reduce the cost per run while maintaining high confidence in the agent's behavior. This involves analyzing recent changes, using statistical sampling for unchanged parts of the codebase, and adapting the testing strategy based on historical data.

#### Common Mistakes
and Follow-Ups
Over-Simplification: Relying solely on random sampling without considering recent changes.
Inadequate Thresholds: Not setting appropriate thresholds for triggering full-suite runs can lead to false positives or false negatives.
Lack of Feedback Mechanism: Failing to incorporate a feedback loop that continuously improves the testing strategy.

To validate our solution, we can set up a dry-run with sample data from recent PRs and monitor the system’s performance over time.

**Comparison:**

```
Category	Relevant Files	Sampled Tests
file_processing	processor.py, reader.py	test_processor_functionality, test_reader_performance
data_storage	storage_manager.py	test_storage_operations (10%)
```

**Code:**

```python
# Example Python code for dynamic test selection based on Git diff output

def get_changed_files(pr_diff_output):
    # Parse Git diff output to identify changed files
    pass

def select_tests(changed_files, full_suite_tests):
    relevant_tests = []
    for file in changed_files:
        for category in full_suite_tests:
            if file in category['relevant_files']:
                relevant_tests += category['tests']
    return relevant_tests

def random_sample_tests(tests, sample_size=10):
    import random
    sampled_tests = random.sample(tests, min(len(tests), sample_size))
    return sampled_tests
```

**Example input / output:**

```text
Input:
- PR diff output indicating changes in processor.py and storage_manager.py

Output:
- Selected tests based on changed files
  - file_processing category: test_processor_functionality, test_reader_performance
  - data_storage category: test_storage_operations (random sample of 10%)
```

**Data:**

```json
{
  "full_suite_tests": [
    {
      "category": "file_processing",
      "relevant_files": ["processor.py", "reader.py"],
      "tests": ["test_processor_functionality", "test_reader_performance"]
    },
    {
      "category": "data_storage",
      "relevant_files": ["storage_manager.py"],
      "tests": ["test_storage_operations"]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Identify Changed Files
4. Are files in relevant categories?
5. Select Relevant Tests
6. Random Sample RemainingTests
7. Run Selected Tests
8. Run Random Sampled Tests
9. Monitor and Feedback

