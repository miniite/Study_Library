---
type: "practice"
difficulty: "Deep"
---

# Design a policy for what code the AI is allowed to touch autonomously versus what requires it to only suggest. Where do you draw the line and why?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Handling AI Inconsistency](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Handling_AI_Inconsistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a policy for when an AI-assisted coding system like Copilot can make autonomous changes, we need to balance between enabling productivity gains while maintaining code quality and consistency. The key is to define clear boundaries around what the AI should autonomously edit versus only suggest modifications.

Firstly, let's triage: The question asks us to design a policy for when Copilot can autonomously modify code versus just suggesting changes. This involves understanding the system’s capabilities, identifying risky areas of the codebase, and setting up guardrails to prevent unintended damage or degradation in code quality. We'll consider factors such as context-awareness, safety, performance, and maintainability.

Drawing the Line
Context-Awareness Limitations: Copilot should only autonomously edit parts of the code where it can reliably understand the context. This typically includes well-documented public APIs, straightforward loops, and simple functions with clear parameters.
Safety Concerns: Certain critical sections of the codebase (e.g., security-sensitive areas, complex business logic) must be protected to prevent accidental changes that could introduce bugs or vulnerabilities.
Performance Impact: Copilot should avoid making autonomous edits in high-performance-critical code where subtle optimizations might have significant impact on execution speed.

#### Cons
istency and Maintainability: Autonomous edits are more suitable for parts of the codebase that follow established patterns, ensuring uniformity across similar sections.

#### First-Principles Solution
Code Context Analysis: Implement a context-aware system that can analyze the code snippet before making an autonomous edit. This includes checking if the function or method has well-defined parameters and clear documentation.
Risk Assessment Model: Develop a risk assessment model based on factors like complexity, criticality, and developer intent. Higher-risk areas (like security functions) should have more stringent checks.
Suggestion-Only Zones: Define specific regions of the codebase that require only suggestions from Copilot. These could include:
Sensitive configurations or settings files
Critical safety-critical components
Highly optimized performance-sensitive sections
Interactive Decision Points: Introduce interactive decision points where the developer must explicitly confirm autonomous edits, ensuring they are aware of any changes being made.
Automated Testing Integrations: Ensure that Copilot's autonomous edits trigger immediate automated testing to catch any unintended side effects early.

#### Closing Summary
and Common Mistakes

In summary, we need a comprehensive policy that leverages Copilot’s capabilities while maintaining robust safeguards. The key is to identify and protect critical code segments from autonomous editing, ensuring that any changes are transparent and reviewed by the developer. Common mistakes include underestimating the complexity of the decision-making process or over-relying on Copilot's suggestions without proper oversight.

Common pitfalls:

Failing to establish clear boundaries between suggestion-only zones and autonomously editable areas.
Ignoring potential security risks associated with autonomous edits in sensitive code segments.
Overly restrictive policies that hinder productivity gains from AI assistance.

**Comparison:**

```
tableRiskAssessment {
    header | Snippet | Complexity Score | Criticality Score | Total Risk
    row | def add(a, b): return a + b | 1 | 1 | 1
    row | def update_security_settings(): set_password('admin', 'secret') | 3 | 5 | 4
}
```

**Code:**

```python
# Example of a function for context-awareness
def is_context_safe(code_snippet):
    # Simple heuristic to check if the snippet is simple and well-documented
    return "## Parameters" in code_snippet and "## Returns" in code_snippet

# Risk assessment model
def assess_risk(snippet):
    complexity_score = calculate_complexity(snippet)
    criticality_score = determine_criticality(snippet)
    return (complexity_score + criticality_score) / 2

def determine_criticality(snippet):
    if "security" in snippet.lower():
        return 5
    elif "performance" in snippet.lower():
        return 4
    return 1

# Example of an interactive prompt
def get_user_confirmation(suggestion, code_snippet):
    print(f"Suggested change: {suggestion}")
    answer = input("Confirm? [y/n]: ")
    if answer.lower() == 'y':
        apply_change(code_snippet)
```

**Example input / output:**

```text
Input:
- Code snippet: def add(a, b): return a + b

Output:
- Risk assessment result: Complexity Score = 1, Criticality Score = 1, Total Risk = 1

Input:
- Code snippet: def update_security_settings(): set_password('admin', 'secret')

Output:
- Risk assessment result: Complexity Score = 3, Criticality Score = 5, Total Risk = 4
```

**Data:**

```yaml
# Example risk assessment results
risk_assessment:
  - snippet: "def add(a, b): return a + b"
    complexity_score: 1
    criticality_score: 1
    total_risk: 1

  - snippet: "def update_security_settings(): set_password('admin', 'secret')"
    complexity_score: 3
    criticality_score: 5
    total_risk: 4
```

**Flow:**

1. Yes
2. No
3. Yes
4. Receive Code Snippet
5. Is Context Safe?
6. Analyze Complexity &Criticality
7. Total Risk High?
8. Autonomous Edit Allowed
9. Interactive Prompt
10. User Confirms
11. Apply Change
12. User Denies
13. Dont Apply Change

