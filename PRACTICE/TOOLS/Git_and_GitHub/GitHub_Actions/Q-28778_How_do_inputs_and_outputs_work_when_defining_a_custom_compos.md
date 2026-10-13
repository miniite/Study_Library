---
type: "practice"
difficulty: "Deep"
---

# How do inputs and outputs work when defining a custom composite action?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Workflow Fundamentals](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Workflow_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how inputs and outputs work in GitHub Actions, especially in custom composite actions, let's break down the concept from first principles. Inputs allow you to pass data into your action so that it can be configured dynamically, while outputs enable your action to return values back to the workflow or other steps within the same action.

#### Triage

GitHub Actions use inputs and outputs through a YAML configuration file. We need to explore how these work in both simple actions and composite actions. Composite actions are reusable groups of commands that encapsulate logic for more complex tasks, making your workflows modular and easier to maintain. Inputs allow us to parameterize these actions, making them versatile, while outputs can be used to pass information from the action back up the call stack or between actions.

#### First-Principles Solution

Defining Inputs:

You define inputs using the inputs section in your action's YAML configuration.
YAML
inputs:
  input-name:
    description: 'A brief description of what this input does.'
    required: true  # Optional, default is false if not specified.
    default: ''    # Provide a default value if no input is provided.

#### Cons
uming Inputs in the Action:

Inside your action script, you can access inputs using ${{ github.action_inputs.<input-name> }}.
BASH
echo "Input Value: ${{ github.action_inputs.input-name }}"

Defining Outputs:

You define outputs in a similar way to inputs but under the outputs section.
YAML
outputs:
  output-name:
    description: 'A brief description of what this output does.'

#### Cons
uming Outputs in the Workflow or Other Actions:

Outputs are automatically available as environment variables in the next steps within the same workflow, and can be passed to other actions using needs and strategy.
BASH
echo "Output Value: ${{ steps.some-action.outputs.output-name }}"

#### Closing Summary
+ Common Mistakes/Follow-Ups

Understanding inputs and outputs is crucial for creating reusable and dynamic GitHub Actions. The key points are:

Inputs allow you to pass configuration data into your action.
Outputs enable the action to provide useful information back to its caller or subsequent actions.

#### Common mistakes
include not properly defining and handling input types, leading to runtime errors, or failing to return relevant outputs from actions. Follow-ups could involve discussing best practices for error handling within custom composite actions or exploring more advanced use cases of inputs and outputs in complex workflows.

**Comparison:**

```
Inputs	Outputs
input-name	output-name
```

**Code:**

```yaml
# Example of a GitHub Action with inputs and outputs

name: Custom Composite Action

on:
  # Define when the action should run
  workflow_call:

jobs:
  example-job:
    runs-on: ubuntu-latest

    steps:
      - name: Step 1
        id: step1
        uses: ./custom-action
        with:
          input-name: "example-value"

      - name: Step 2
        id: step2
        run: |
          echo "Step 1 output value is ${{ steps.step1.outputs.output-name }}"
```

**Example input / output:**

```text
# Step 1 - Running the custom action with an input value
Step 1 output: Input Value: example-value

# Step 2 - Consuming the output from step 1 in a subsequent step
Output Value: example-output-value
```

**Data:**

```yaml
# Custom Action YAML Configuration
name: custom-action
description: A reusable action with inputs and outputs

inputs:
  input-name:
    description: 'A brief description of what this input does.'
    required: true
    default: ''

outputs:
  output-name:
    description: 'A brief description of what this output does.'
```

**Flow:**

1. Workflow
2. Call Action
3. Action with Inputs
4. Process Input
5. Generate Output
6. Pass Output to Next Step

