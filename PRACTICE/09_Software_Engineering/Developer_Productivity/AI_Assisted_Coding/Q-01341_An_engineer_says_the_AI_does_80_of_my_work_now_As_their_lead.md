---
type: "practice"
difficulty: "Deep"
---

# An engineer says the AI 'does 80% of my work now'. As their lead, how do you validate that claim and what risks does it raise for the team's long-term skill?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Measuring AI Productivity](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Measuring_AI_Productivity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

As a Fullstack SDE-2/3 focused on backend development, the statement that "the AI 'does 80% of my work now'" requires careful validation to ensure it doesn't obscure critical aspects of our team's long-term skills and productivity. To validate this claim and assess the associated risks, we need to break down what constitutes the engineer’s daily tasks and how much automation can truly handle.

The core idea is that while AI-assisted coding tools like Copilot can significantly speed up development by generating boilerplate code, refactoring existing code, and suggesting optimizations, they cannot replace human judgment and creativity in complex problem-solving. Let's delve into this step-by-step.

#### Triage
What the Question Is Asking
What is the Engineer’s Claim? - The engineer claims that 80% of their work is being handled by AI.
Validation Methodology - We need to assess the accuracy and reliability of this claim.
Risk Assessment - Identify potential risks for the team's long-term skillset.

#### Assumptions
The term "work" includes coding, debugging, problem-solving, writing tests, documentation, and maintaining code quality.
AI tools like Copilot can automate some parts but not all aspects of development.
The engineer’s primary role is backend development, though the principles apply to fullstack roles as well.
Step-by-Step Solution

Code Audit - Review a sample of recent projects or commits where the AI was used extensively. Look for patterns and common tasks that were handled by Copilot.

Manual vs. Automated Tasks - Categorize the tasks performed into automated (handled by Copilot) and manual (still done manually). For example:

Code generation
Refactoring suggestions
Documentation updates
Debugging assistance
Complex problem-solving

Impact on Skill Development - Assess how much of the hands-on experience in complex problem-solving, understanding system architecture, and debugging is being replaced by AI.

Performance Metrics - Compare metrics such as code quality, bug rate, productivity, and overall project timelines before and after the introduction of AI tools.

Detailed Breakdown

Code Audit

Code Generation
Copilot
Refactoring Suggestions
Copilot
Documentation Updates
Copilot
Debugging Assistance
Copilot
Complex Problem-Solving
Manual Effort

Manual vs. Automated Tasks

Code Generation: 60% automated, 40% manual.
Refactoring Suggestions: 50% automated, 50% manual (evaluation and application).
Documentation Updates: 80% automated, 20% manual.
Debugging Assistance: 30% automated, 70% manual.
Complex Problem-Solving: 20% automated, 80% manual.

#### Closing Summary

By breaking down the tasks and reviewing both manual and automated contributions, we can get a clearer picture of how much AI is indeed doing "80% of the work." The key risks are:

Reduced opportunities for developers to develop deep technical skills.
Potential complacency in learning and mastering complex problem-solving techniques.
Dependency on AI tools leading to loss of critical thinking abilities.

#### Common Mistakes/Follow-Ups
Mistake 1: Over-reliance on AI without understanding its limitations. For instance, AI might not handle edge cases or unusual scenarios effectively.
Mistake 2: Ignoring the importance of hands-on experience and problem-solving skills.

#### Follow-Up
Questions:
How can we ensure that developers are still actively learning and improving their skills?
What measures can be taken to balance AI assistance with human judgment?

**Comparison:**

| Task Type         | Percentage Automated | Percentage Manual |
|-------------------|----------------------|------------------|
| Code Generation   | 60%                  | 40%              |
| Refactoring       | 50%                  | 50%              |
| Documentation     | 80%                  | 20%              |
| Debugging         | 30%                  | 70%              |
| Problem-Solving   | 20%                  | 80%              |

**Code:**

```python
def validate_ai_assistance(code_samples):
    # Placeholder function to simulate a code audit
    automated_tasks = {
        "code_generation": 60,
        "refactoring_suggestions": 50,
        "documentation_updates": 80,
        "debugging Assistance": 30,
        "complex_problem-solving": 20
    }

    total_tasks = sum(automated_tasks.values())
    print(f"Total tasks: {total_tasks}")
    for task, percentage in automated_tasks.items():
        print(f"{task}: {percentage}%")
```

**Example input / output:**

| File Path | Code Generation | Refactoring Suggestions | Documentation Updates | Debugging Assistance | Complex Problem-Solving |
|-----------|-----------------|------------------------|----------------------|---------------------|-----------------------|
| src/handlers/user.py | 60% | 50% | 80% | 30% | 20% |
| tests/unit_tests/test_user.py | 45% | 60% | 15% | 20% | 70% |

**Data:**

```json
{
  "code_samples": [
    {
      "file_path": "src/handlers/user.py",
      "automated_tasks_count": {
        "code_generation": 60,
        "refactoring_suggestions": 50,
        "documentation_updates": 80,
        "debugging_assistance": 30,
        "complex_problem_solving": 20
      }
    },
    {
      "file_path": "tests/unit_tests/test_user.py",
      "automated_tasks_count": {
        "code_generation": 45,
        "refactoring_suggestions": 60,
        "documentation_updates": 15,
        "debugging_assistance": 20,
        "complex_problem_solving": 70
      }
    }
  ]
}
```

**Flow:**

1. Code Audit
2. Manual vs. Automated Tasks
3. Validate AI Assistance
4. Impact on Skill Development
5. Performance MetricsComparison
6. Mitigation Strategies

