---
type: "practice"
difficulty: "Deep"
---

# Two engineers edited the same agent prompt on different branches, both merged with no textual conflict, yet the agent's behavior broke. Explain this semantic conflict and what CI gate would have caught it.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Merge Conflicts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Merge_Conflicts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a distributed software development environment like GitHub, where multiple engineers are working on separate branches of the same project, conflicts can arise not just at the level of code, but also in terms of semantic understanding or intended behavior. This scenario perfectly illustrates such a situation. Engineers edited the same agent prompt independently and both changes were merged without any textual conflict, yet the final outcome caused unexpected behavior. To understand this, we need to explore how semantic conflicts can occur and what CI (Continuous Integration) gate could have caught it.

To break down the problem:

Semantic Conflict: This occurs when multiple engineers interpret or modify the same codebase in a way that leads to unintended interactions between their changes.
CI Gate Requirement: A robust CI system should catch semantic conflicts by verifying not just syntactic correctness but also logical consistency and functional behavior.

#### Triage

The question is probing our understanding of:

How semantic issues can arise during parallel development.
The importance of ensuring that merged code behaves as intended.
What tools or practices can help prevent such issues in a CI pipeline.

#### Assumptions
include:

Engineers are working on different branches and have independently made changes to the same agent prompt.
There is no textual conflict, meaning their changes do not overlap directly but interact indirectly through the agent's behavior.

#### First Principles Solution

Let’s start by considering how an agent prompt might be structured. Assume a simple prompt format with variables that are dynamically populated based on context:

PYTHON
def generate_prompt(user_input):
    return f"Prompt: {user_input} | Action: {action}"

Now, let's say Engineer A and Engineer B both made changes independently:

Engineer A updates the action logic to handle specific inputs differently.
Engineer B adds a new condition in the agent response that should be executed with certain user inputs.

Without any direct textual conflict, their changes could interact in ways not anticipated. For example:

Engineer A’s update: if input == "specific_condition": action = "handle_special_case"
Engineer B’s addition: if input.startswith("prefix"): return f"Special prompt for {input}

Both engineers merge independently without realizing that these new conditions could conflict or cause unexpected behavior.

CI Gate Requirement

To catch such semantic conflicts, a robust CI pipeline should include the following checks:

Unit Tests: Ensure each change works as intended in isolation.
Integration Tests: Verify that changes integrate well with other parts of the system without breaking existing functionality.
End-to-End Tests: Test the entire workflow from user input to agent response to ensure consistent behavior.

For example, an end-to-end test might look like this:

PYTHON
def test_agent_response():
    prompt = generate_prompt("specific_condition")
    assert "handle_special_case" in prompt

    prompt = generate_prompt("prefix_value")
    assert "Special prompt for prefix_value" in prompt

    # Test both conditions together to ensure they don’t conflict.
    input_data = {"user_input": "specific_prefix", "context": {}}
    response = agent_handle(input_data)
    expected_response = f"Handle special case for specific_prefix | Action: handle_special_case"
    assert response == expected_response

#### Closing Summary
and Common Mistakes/Follow-Ups

This scenario highlights the importance of thorough testing, especially integration and end-to-end tests. A CI pipeline that includes these checks would have caught this semantic conflict before merging.

#### Common mistakes
include:

Not having a comprehensive test suite.
Assuming that individual changes are sufficient without considering how they interact with other parts of the system.
Over-reliance on manual testing which is often inefficient and prone to human error.

#### Follow-ups
might include discussing how to prioritize tests, automate them, or even use more advanced tools like static code analysis to catch potential semantic conflicts early in the development process.

