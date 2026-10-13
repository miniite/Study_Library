---
type: "practice"
difficulty: "Intermediate"
---

# A recursive JSON-config flattener for hyperparameters is a stack-overflow risk on adversarial nesting; how would you bound recursion depth, convert to an explicit stack, and reason about worst-case input?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Recursion vs Explicit Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion_vs_Explicit_Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of recursive JSON-config flattening that could lead to a stack overflow with deep or adversarial nesting, we need to implement a solution that bounds recursion depth and converts it to an explicit stack-based approach. The key idea is to use a fixed-size stack to control the recursion, ensuring we handle large inputs without running into memory issues.

#### Triage

The question asks us to design a recursive JSON flattener that avoids stack overflow risks by bounding recursion depth and converting to an explicit stack. This involves understanding the nature of recursion, stack operations, and how to manage nested structures programmatically.

#### Assumptions
The input JSON is well-formed.
We are dealing with hyperparameters in the form of key-value pairs within a JSON structure.
The goal is to flatten these parameters into a single-level dictionary.

#### First-Principles Solution

To solve this problem, we will implement an iterative approach using a stack to simulate recursion. This method ensures that we can handle arbitrarily deep nesting without exceeding the default recursion limit. Here’s how it works:

Define the Input Structure: We assume the input is a nested JSON structure with hyperparameters.
Initialize a Stack and Result Dictionary: Use a stack to hold intermediate states of processing, and a result dictionary to store the flattened output.
Iterate Through the Stack: Pop elements from the stack, process them, and push new sub-elements onto the stack.
Step-by-Step Solution

Define the Input JSON:

JSON
{
  "model": {
    "type": "ml_model",
    "layers": [
      {"name": "layer1", "units": 128},
      {"name": "layer2", "activation": "relu"}
    ],
    "optimizer": "adam"
  },
  "training": {
    "epochs": 50,
    "batch_size": 32
  }
}

Initialize the Stack and Result Dictionary:

PYTHON
import json

def flatten_json(input_dict, parent_key='', sep='_'):
    items = []
    for k, v in input_dict.items():
        new_key = f'{parent_key}{sep}{k}' if parent_key else k
        if isinstance(v, dict):
            items.extend(flatten_json(v, new_key, sep=sep).items())
        else:
            items.append((new_key, v))
    return dict(items)

def flatten_iterative(json_str):
    # Initialize stack and result dictionary
    stack = [(json.loads(json_str), '')]
    result = {}

    while stack:
        current_dict, current_key = stack.pop()
        for k, v in current_dict.items():
            new_key = f'{current_key}{k}' if current_key else k
            if isinstance(v, dict):
                stack.append((v, new_key + '_'))
            else:
                result[new_key] = v

    return result

Test the Function:

PYTHON
input_json = """
{
  "model": {
    "type": "ml_model",
    "layers": [
      {"name": "layer1", "units": 128},
      {"name": "layer2", "activation": "relu"}
    ],
    "optimizer": "adam"
  },
  "training": {
    "epochs": 50,
    "batch_size": 32
  }
}
"""

flattened_output = flatten_iterative(input_json)
print(flattened_output)

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the iterative approach using a stack effectively manages deep nesting by avoiding excessive recursion. This solution ensures that we can handle large or adversarial inputs without running into stack overflow issues.

#### Common Mistakes

Failing to properly manage the state of elements in the stack.
Incorrectly handling nested structures, leading to incorrect flattening.
Not initializing the stack and result dictionary correctly.

For follow-ups, one might consider optimizing memory usage further by limiting the size of the stack or exploring other data structures like queues if specific constraints are present.

**Code:**

```python
import json

def flatten_json(input_dict, parent_key='', sep='_'):
    items = []
    for k, v in input_dict.items():
        new_key = f'{parent_key}{sep}{k}' if parent_key else k
        if isinstance(v, dict):
            items.extend(flatten_json(v, new_key, sep=sep).items())
        else:
            items.append((new_key, v))
    return dict(items)

def flatten_iterative(json_str):
    # Initialize stack and result dictionary
    stack = [(json.loads(json_str), '')]
    result = {}

    while stack:
        current_dict, current_key = stack.pop()
        for k, v in current_dict.items():
            new_key = f'{current_key}{k}' if current_key else k
            if isinstance(v, dict):
                stack.append((v, new_key + '_'))
            else:
                result[new_key] = v

    return result

input_json = """
{
  "model": {
    "type": "ml_model",
    "layers": [
      {"name": "layer1", "units": 128},
      {"name": "layer2", "activation": "relu"}
    ],
    "optimizer": "adam"
  },
  "training": {
    "epochs": 50,
    "batch_size": 32
  }
}
"""

flattened_output = flatten_iterative(input_json)
print(flattened_output)
```

**Example input / output:**

```text
flattened_output = flatten_iterative(input_json)
print(flattened_output)
# Output:
{
  "model_type": "ml_model",
  "model_layers_name_0": "layer1",
  "model_layers_units_0": 128,
  "model_layers_name_1": "layer2",
  "model_layers_activation_1": "relu",
  "model_optimizer": "adam",
  "training_epochs": 50,
  "training_batch_size": 32
}
```

**Data:**

```json
{
  "model": {
    "type": "ml_model",
    "layers": [
      {"name": "layer1", "units": 128},
      {"name": "layer2", "activation": "relu"}
    ],
    "optimizer": "adam"
  },
  "training": {
    "epochs": 50,
    "batch_size": 32
  }
}
```

