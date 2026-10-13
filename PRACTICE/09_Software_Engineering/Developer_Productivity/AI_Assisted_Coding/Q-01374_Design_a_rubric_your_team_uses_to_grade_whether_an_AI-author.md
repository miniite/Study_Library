---
type: "practice"
difficulty: "Deep"
---

# Design a rubric your team uses to grade whether an AI-authored change is 'ready to merge' independent of who or what wrote it.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Measuring AI Productivity](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Measuring_AI_Productivity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a rubric for grading whether an AI-authored change is "ready to merge," we need to focus on multiple dimensions including code quality, performance, maintainability, and adherence to coding standards. The core idea is to establish clear criteria that can be objectively assessed, ensuring the merged code meets our high standards regardless of its origin.

The rubric should cover various aspects:

Code Quality: Assessing whether the code follows best practices, is well-structured, and is free from obvious bugs.
Performance: Evaluating if the change introduces any performance issues or optimizations are made where needed.
Maintainability: Ensuring that the code can be easily maintained by other developers in the future.
Security: Checking for potential security vulnerabilities introduced by the AI.

Let's dive into each of these areas from first principles:

Code Quality
Syntax and Formatting: Ensure the code adheres to our coding standards, including line length, indentation, and naming conventions.
Correctness: Verify that the logic is correct and does not introduce bugs or logical errors.
Readability: The code should be easy to read and understand by other developers.
Performance
Efficiency: Check if the AI-generated code uses efficient algorithms and data structures.
Resource Usage: Ensure there are no unnecessary resource consumption, such as excessive memory usage or CPU load.
Maintainability
Modularity: The code should be modular and easy to understand.
Documentation: Adequate comments and documentation are provided for the new additions.

#### Cons
istency with Existing Codebase: New code should align with existing coding patterns and practices.
Security
Input Validation: Ensure all user inputs are validated properly.
Sensitive Data Handling: Check that sensitive data is handled securely, such as proper encryption or use of secure APIs.

To implement this rubric, we can break it down into a structured checklist:

MARKDOWN
| Category       | Criteria                                    | Weight |
|----------------|---------------------------------------------|--------|
| Code Quality   | Syntax and formatting                       | 20%    |
|                | Correctness                                 | 30%    |
|                | Readability                                 | 15%    |
| Performance    | Efficiency                                  | 20%    |
|                | Resource usage                              | 20%    |
| Maintainability| Modularity                                  | 15%    |
|                | Documentation                               | 10%    |
|                | Consistency with existing codebase          | 10%    |
| Security       | Input validation                            | 10%    |
|                | Sensitive data handling                      | 10%    |

Each category has a defined set of criteria and associated weights, allowing us to assign scores based on the extent to which each criterion is met. A score below a certain threshold would indicate that the code needs further review or refinement.

#### Closing Summary

By designing such a rubric, we ensure that AI-generated changes are thoroughly vetted before merging into our main repository. This approach guarantees consistent quality and reliability, regardless of whether the change was authored by an AI system like Copilot or a human developer.

#### Common mistakes
to avoid include:

Relying too heavily on automated tools without manual review.
Overlooking edge cases that might not be covered in standard tests.
Failing to update the rubric as our project evolves and new technologies are introduced.

**Comparison:**

```
Category	Criteria	Weight
Code Quality	Syntax and formatting	20%
	Correctness	30%
	Readability	15%
Performance	Efficiency	20%
	Resource usage	20%
Maintainability	Modularity	15%
	Documentation	10%
	Consistency with existing codebase	10%
Security	Input validation	10%
	Sensitive data handling	10%
```

**Code:**

```
MARKDOWN
Copy
| Category       | Criteria                                    | Weight |
|----------------|---------------------------------------------|--------|
| Code Quality   | Syntax and formatting                       | 20%    |
|                | Correctness                                 | 30%    |
|                | Readability                                 | 15%    |
| Performance    | Efficiency                                  | 20%    |
|                | Resource usage                              | 20%    |
| Maintainability| Modularity                                  | 15%    |
|                | Documentation                               | 10%    |
|                | Consistency with existing codebase          | 10%    |
| Security       | Input validation                            | 10%    |
|                | Sensitive data handling                      | 10%    |
```

**Example input / output:**

```text
Example Input:
- Code change authored by AI

Expected Output:
- Rubric score for each category and overall grade
- Detailed feedback on areas that need improvement
```

**Data:**

```json
{
  "rubric": [
    {
      "category": "Code Quality",
      "criteria": ["Syntax and formatting", "Correctness", "Readability"],
      "weight": [20, 30, 15]
    },
    {
      "category": "Performance",
      "criteria": ["Efficiency", "Resource usage"],
      "weight": [20, 20]
    },
    {
      "category": "Maintainability",
      "criteria": ["Modularity", "Documentation", "Consistency with existing codebase"],
      "weight": [15, 10, 10]
    },
    {
      "category": "Security",
      "criteria": ["Input validation", "Sensitive data handling"],
      "weight": [10, 10]
    }
  ]
}
```

**Flow:**

1. RubricCheck
2. Yes
3. No
4. Code Authorship
5. Is AI-authored?
6. Rubric Check
7. Merge if Passes Rubric
8. Rubric Check
9. Syntax and formatting
10. Correctness
11. Readability
12. Efficiency
13. Resource usage
14. Modularity
15. Documentation
16. Consistency with existingcodebase
17. Input validation
18. Sensitive data handling

