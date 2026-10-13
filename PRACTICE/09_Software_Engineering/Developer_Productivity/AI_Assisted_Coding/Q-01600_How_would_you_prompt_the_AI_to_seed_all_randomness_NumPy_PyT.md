---
type: "practice"
difficulty: "Intermediate"
---

# How would you prompt the AI to seed all randomness (NumPy, PyTorch, Python) for reproducibility, and confirm results actually repeat?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Verifying AI Generated ML Code](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Verifying_AI_Generated_ML_Code.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure reproducibility in machine learning experiments involving NumPy, PyTorch, and Python's built-in random module, we need to seed all sources of randomness consistently across our codebase. This involves setting seeds for each library or framework used, and then verifying that the same sequence of results is obtained when running the code multiple times.

Firstly, let’s break down what needs to be done:

Set the NumPy random seed.
Set the PyTorch random seed.
Seed Python's built-in random module.
Verify that the operations produce the same output upon rerunning the code.

Here’s how we can achieve this using a simple and effective prompt for AI-assisted coding tools like Copilot:

PYTHON
# Prompt for AI-assisted coding (Copilot/Cursor)
# Step 1: Set NumPy random seed
import numpy as np

np.random.seed(42)

# Step 2: Set PyTorch random seed
import torch

torch.manual_seed(42) # For CPU operations
torch.cuda.manual_seed_all(42) # For GPU operations, if applicable

# Step 3: Seed Python's built-in random module
import random

random.seed(42)

# Step 4: Perform some random operation and print the results for reproducibility check
a = np.random.rand()
b = torch.randn((2, 2))
c = random.random()

print("NumPy Random Number:", a)
print("PyTorch Tensor:", b)
print("Python's built-in random number:", c)

# Save these values to a file or compare them later to ensure reproducibility

To further solidify our understanding and implementation, let’s break down the steps in more detail:

Setting NumPy Random Seed:

We use np.random.seed(42) which ensures that all operations involving NumPy will generate the same sequence of random numbers each time we run the code.

Setting PyTorch Random Seed:

For CPU, we set torch.manual_seed(42).
For GPU, if using multiple GPUs or a single CUDA-capable GPU, we use torch.cuda.manual_seed_all(42) to ensure all threads generate the same sequence of random numbers.

Seeding Python's Built-in Random Module:

We use random.seed(42), which seeds the generator used by functions like rand() and others.

Verifying Reproducibility:

By running the code multiple times, we should observe that the output values of a, b, and c remain consistent.

To test this, we can execute the code above and then re-run it to verify if all outputs match the initial runs.

**Code:**

```python
# Prompt for AI-assisted coding (Copilot/Cursor)
import numpy as np

np.random.seed(42)

torch.manual_seed(42) # For CPU operations
torch.cuda.manual_seed_all(42) # For GPU operations, if applicable

random.seed(42)

a = np.random.rand()
b = torch.randn((2, 2))
c = random.random()

print("NumPy Random Number:", a)
print("PyTorch Tensor:", b)
print("Python's built-in random number:", c)
```

**Example input / output:**

```text
NumPy Random Number: 0.5488135039273248
PyTorch Tensor:
tensor([[ 0.7614, -1.4196],
        [ 1.1258, -1.0356]])
Python's built-in random number: 0.768523866092527
```

**Flow:**

1. Set NumPy Random Seed
2. Run Code
3. Set PyTorch Random Seed(CPU)
4. Set PyTorch Random Seed(GPU)
5. Set Python's Built-in RandomModule
6. Execute Code and PrintOutputs

