---
type: "practice"
difficulty: "Basic"
---

# You paste a long traceback from a failed model.fit into Cursor. What should you trim or add so it actually helps rather than overwhelms?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Prompting for Production Patterns](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Production_Patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When pasting a long traceback from a failed model.fit() operation into AI-assisted coding tools like GitHub Copilot, the key is to focus on the most critical information while making it easy for the tool to understand and provide relevant suggestions. Here’s how you can trim or add necessary details:

TL;DR

Trim the traceback to include only the last few error messages that indicate the issue, along with any preceding context like import statements or configuration settings. Add comments and brief explanations where needed to guide the AI.

#### Triaging the Question

The core of this question is about ensuring that you provide a minimal but informative snippet that the AI tool can understand and help with efficiently. The interviewer might be probing your understanding of:

Relevance: What parts of the traceback are relevant versus noise.
Context: How much surrounding context (like imports or configurations) is necessary to provide a clear picture.
Clarity: The importance of adding comments for clarity and guidance.
Reasoning from First Principles
Step 1: Identify the Problem

First, identify the last few error messages in the traceback that indicate what went wrong. These are typically at the bottom of the traceback.

Step 2: Trim Irrelevant Parts

Remove any parts of the traceback that do not directly lead to understanding and fixing the issue. This includes:

Long sections before the first error message.
Duplicates or repeated messages.
Stack traces from other unrelated functions.

For example, if your last few lines look like this:

CODE
  File "/path/to/your/script.py", line 123, in <module>
    model.fit(X_train, y_train)
  File "/opt/python/lib/python3.8/site-packages/tensorflow/python/training/tracking/tracker.py", line 405, in fit
    return self._method(self, x, *args, **kwargs)
  File "/opt/python/lib/python3.8/site-packages/tensorflow/python/keras/engine/training_v2.py", line 1169, in _method
    output_loss = self.compute_loss(
  ...

You can trim it to focus on the last few lines:

CODE
  File "/path/to/your/script.py", line 123, in <module>
    model.fit(X_train, y_train)
ValueError: X and y should have same first dimension, got X.shape = (20,) and y.shape = (4,)
Step 3: Add Context

Include any necessary context that might help the AI understand the situation better. This could be:

Relevant import statements.
Key configuration parameters or hyperparameters.
Any recent changes to the code.

For instance, if your script looks like this:

PYTHON
import tensorflow as tf

# Define model architecture
model = tf.keras.models.Sequential([...])

# Load data
X_train, y_train = load_data()

# Fit the model (this is where it fails)
model.fit(X_train, y_train)

def load_data():
    # Some data loading logic here...

You might add a brief comment to indicate that you're trying to debug an issue with fitting the model:

PYTHON
import tensorflow as tf

# Define model architecture
model = tf.keras.models.Sequential([...])

# Load data
X_train, y_train = load_data()

# Fit the model (this is where it fails)
model.fit(X_train, y_train)  # ValueError: X and y should have same first dimension, got X.shape = (20,) and y.shape = (4,)

def load_data():
    # Some data loading logic here...
Step 4: Provide Additional Guidance

If the issue is related to specific operations or configurations, you might add a comment explaining what’s going wrong. For example:

PYTHON
import tensorflow as tf

# Define model architecture
model = tf.keras.models.Sequential([...])

# Load data (make sure X and y have compatible shapes)
X_train, y_train = load_data()

# Fit the model (this is where it fails)
model.fit(X_train, y_train)  # ValueError: X and y should have same first dimension

def load_data():
    # Some data loading logic here...

#### Closing Summary

By carefully trimming down the traceback to include only the critical information and adding context and explanations, you ensure that AI tools like Copilot can understand the issue quickly and provide relevant suggestions. This approach helps in efficiently debugging your code without overwhelming the tool with unnecessary details.

#### Common Mistakes/Follow-Ups
Over-trimming: Not including enough context to help the AI understand the situation.
Lack of Comments: Not adding comments or explanations can make it harder for the AI to provide useful suggestions.
Including Too Much Data: Including too much irrelevant data can confuse the AI and lead to less effective suggestions.

**Code:**

```python
import tensorflow as tf

# Define model architecture
model = tf.keras.models.Sequential([...])

# Load data (make sure X and y have compatible shapes)
X_train, y_train = load_data()

# Fit the model (this is where it fails)
model.fit(X_train, y_train)  # ValueError: X and y should have same first dimension

def load_data():
    # Some data loading logic here...
```

