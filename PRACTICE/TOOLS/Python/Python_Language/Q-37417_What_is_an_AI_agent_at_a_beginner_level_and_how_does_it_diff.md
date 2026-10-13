---
type: "practice"
difficulty: "Basic"
---

# What is an AI agent at a beginner level, and how does it differ from a single LLM call in a Python script?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming LLM Responses](<../../../../TOOLS/Python/Python_Language/Streaming_LLM_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An AI agent at a beginner level can be thought of as a simple software entity that interacts with its environment to achieve goals or objectives. In contrast, calling an LLM (Language Model) directly from a Python script is more about processing text inputs and generating responses based on those inputs. Let's break down the differences:

#### Triage
What Is Being Probed

The interviewer is probing your understanding of AI agents in general and how they differ from direct API calls to language models like LLMs. They are likely interested in concepts such as autonomy, environment interaction, decision-making, and goal-oriented behavior.

Key Assumptions
We assume the agent operates within a defined environment that can provide information and receive actions.
The environment could be simulated or real-world (e.g., robotics, games).
The LLM is used for generating text-based responses or instructions based on input provided by the agent.
Step-by-Step Solution
1. Understanding AI Agents

An AI agent typically has the following components:

Environment Interaction: It can observe the state of its environment and take actions.
State Representation: The internal representation of the current state of the environment.
Action Selection Mechanism: A mechanism to choose the best action based on the current state.
Goal-Oriented Behavior: Actions are taken with the intention of achieving certain goals.
2. Example of an AI Agent

Here’s a simple example in Python where we simulate an agent that navigates through a grid world and collects items:

PYTHON
import numpy as np

class GridWorldAgent:
    def __init__(self, grid_size):
        self.grid = np.zeros((grid_size, grid_size), dtype=int)
        self.position = (0, 0)  # Initial position in the grid
        self.items_collected = 0

    def move(self, direction):
        x, y = self.position
        if direction == 'up':
            new_y = max(0, y - 1)
        elif direction == 'down':
            new_y = min(y + 1, self.grid.shape[0] - 1)
        elif direction == 'left':
            new_x = max(0, x - 1)
        elif direction == 'right':
            new_x = min(x + 1, self.grid.shape[1] - 1)

        if self.grid[new_y, new_x] > 0:
            self.items_collected += self.grid[new_y, new_x]
            self.grid[new_y, new_x] = 0

        self.position = (new_x, new_y)

    def reset(self):
        self.__init__(self.grid.shape[0])

# Example usage
agent = GridWorldAgent(5)
agent.move('right')
print(agent.position)  # Expected output: (0, 1)
3. Understanding LLM Calls in Python

In contrast, calling an LLM directly is about processing text inputs and generating responses:

PYTHON
import openai

def call_llm(prompt):
    response = openai.Completion.create(
        engine="davinci",
        prompt=prompt,
        max_tokens=1024,
        temperature=0.7,
        n=1,
        stop=None
    )
    return response.choices[0].text.strip()

# Example usage
prompt = "What is the capital of France?"
response = call_llm(prompt)
print(response)  # Expected output: A generated response from the LLM

#### Closing Summary
+ Common Mistakes/Follow-Ups

An AI agent in a broader sense involves more than just processing text. It includes elements like interaction with an environment, state representation, and decision-making based on current states to achieve goals. The direct call to an LLM is simpler and focuses solely on generating text responses.

#### Common Mistakes
Overlooking the Agent's Environment: Sometimes beginners might think of agents as standalone entities without a context or need for interaction.
Misunderstanding the Role of State Representation: Not recognizing that state representation is crucial in decision-making processes.
Assuming LLMs Handle All Aspects: Thinking an LLM can replace all aspects of AI agent behavior, when it's primarily focused on text generation.

#### Follow-Ups
Can you describe a scenario where an AI agent would be more appropriate than an LLM call?
How might you integrate an LLM response within the decision-making process of your AI agent?

**Comparison:**

```
Action	New Position
Move right	(0, 1)
Move up	(1, 0)
Move down	(3, 0)
Move left	(0, -1)
```

**Code:**

```

import numpy as np

class GridWorldAgent: def init(self, grid_size): self.grid = np.zeros((grid_size, grid_size), dtype=int) self.position = (0, 0) # Initial position in the grid self.items_collected = 0

CODE
Copy
def move(self, direction):
    x, y = self.position
    if direction == 'up':
        new_y = max(0, y - 1)
    elif direction == 'down':
        new_y = min(y + 1, self.grid.shape[0] - 1)
    elif direction == 'left':
        new_x = max(0, x - 1)
    elif direction == 'right':
        new_x = min(x + 1, self.grid.shape[1] - 1)

    if self.grid[new_y, new_x] > 0:
        self.items_collected += self.grid[new_y, new_x]
        self.grid[new_y, new_x] = 0

    self.position = (new_x, new_y)

def reset(self):
    self.__init__(self.grid.shape[0])
CODE
Copy

```python
import openai

def call_llm(prompt):
    response = openai.Completion.create(
        engine="davinci",
        prompt=prompt,
        max_tokens=1024,
        temperature=0.7,
        n=1,
        stop=None
    )
    return response.choices[0].text.strip()
```

**Example input / output:**

```text
# Output of initial grid state and position
Initial position: (0, 0)

[[0. 0. 0. 0. 0.]
 [0. 0. 0. 1. 0.]
 [0. 0. 0. 0. 0.]
 [0. 0. 0. 0. 0.]
 [0. 0. 0. 0. 2.]]
```

**Data:**

```python
# Sample grid for the agent
grid = np.zeros((5, 5), dtype=int)
grid[2, 3] = 1
grid[4, 4] = 2

# Example usage of the agent
agent = GridWorldAgent(5)
print("Initial position:", agent.position)  # (0, 0)
print(agent.grid)
```

**Flow:**

1. Agent
2. Environment Interaction
3. State Representation
4. Action Selection Mechanism
5. Goal-Oriented Behavior
6. Observation of State Changes
7. Decision Making Based onState

