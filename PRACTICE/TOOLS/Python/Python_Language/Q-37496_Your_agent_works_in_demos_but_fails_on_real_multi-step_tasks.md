---
type: "practice"
difficulty: "Expert"
---

# Your agent works in demos but fails on real multi-step tasks. How would you build a Python trajectory-evaluation harness that scores whether the agent took the right steps, not just the final answer?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Offline Evaluation Harnesses](<../../../../TOOLS/Python/Python_Language/Safe_User-authored_Configuration/Offline_Evaluation_Harnesses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to design a Python trajectory-evaluation harness that evaluates an AI agent's performance during multi-step tasks, focusing on both the process and the outcome. The key idea is to break down the task into steps and score each step based on correctness, rather than just evaluating the final answer.

#### Triage

The question asks for a method to evaluate an AI agent's progress in complex, multi-step tasks by scoring its behavior at each step. We need to ensure that our evaluation system can handle sequences of actions, provide detailed feedback, and scale appropriately for various tasks. Key assumptions include:

The task is well-defined with clear steps.
The agent’s actions are observable and recorded.

#### Solution
from First Principles
Define Task Steps: Clearly define the sequence of steps required to complete a task.
Record Agent Actions: Log each action taken by the agent during the task.
Step-wise Evaluation Function: Develop an evaluation function that scores each step based on correctness.
Aggregate Scores: Aggregate the scores from each step to determine the overall performance.
Step-by-Step Solution
Define Task Steps:
Identify and list all steps required to complete a task. For example, in a game like Chess, steps might include moving pieces, checking for valid moves, etc.
Record Agent Actions:
Use logging or tracing mechanisms to record the agent's actions at each step. This could involve tracking variables such as move history, state changes, and decisions made.
Step-wise Evaluation Function:
For each defined step, create a function that evaluates whether the agent took the correct action. This might involve comparing the agent’s chosen action to expected actions or using domain-specific heuristics.
Aggregate Scores:
Calculate an overall score by summing up scores from all steps. Adjust weights if certain steps are more critical than others.
Code Implementation

Here's a basic implementation outline:

PYTHON
class TrajectoryEvaluator:
    def __init__(self, task_steps):
        self.task_steps = task_steps  # List of step definitions

    def evaluate(self, agent_actions):
        """Evaluates the trajectory based on recorded actions."""
        total_score = 0.0
        current_step_index = 0

        for action in agent_actions:
            if current_step_index < len(self.task_steps) and self.task_steps[current_step_index]['action'] == action['action']:
                score = self._evaluate_step(action)
                total_score += score
                current_step_index += 1
            else:
                # Penalize incorrect or out-of-order actions
                total_score -= 0.5

        return total_score

    def _evaluate_step(self, step):
        """Evaluates a single step based on the action taken."""
        expected_action = self.task_steps[current_step_index]['action']

        if step['action'] == expected_action:
            return 1.0
        else:
            return -0.5
Sample Usage
PYTHON
# Define task steps for a simple example (like Chess)
task_steps = [
    {'action': 'Move piece A to position B'},
    {'action': 'Check if opponent is in check'}
]

# Agent actions taken during a demo
agent_actions = [
    {'time': 1, 'action': 'Move piece A to position B'},
    {'time': 2, 'action': 'Check if opponent is in check'},
    # More steps here...
]

evaluator = TrajectoryEvaluator(task_steps)
score = evaluator.evaluate(agent_actions)
print(f"Total score: {score}")

#### Closing Summary
and Common Mistakes

This solution provides a structured approach to evaluating AI agents by breaking down tasks into manageable steps. However, common pitfalls include:

Overlooking edge cases where the agent might take valid but non-optimal actions.
Not adequately weighting steps that are more critical or complex.

By addressing these issues, we can develop a robust evaluation harness that not only measures final outcomes but also provides valuable feedback on performance throughout the task.

