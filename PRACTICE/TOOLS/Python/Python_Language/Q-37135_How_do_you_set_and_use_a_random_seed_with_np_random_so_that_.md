---
type: "practice"
difficulty: "Intermediate"
---

# How do you set and use a random seed with np.random so that an experiment is reproducible, and why does reproducibility matter in ML?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Drift-triggered Retraining](<../../../../TOOLS/Python/Python_Language/Python_Tail_Latency/Drift-triggered_Retraining.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure reproducibility in experiments using Python and np.random, setting the random seed is crucial. This ensures that any operations involving randomness (like shuffling data, generating random numbers) produce the same results every time the code is run with the same seed value. Reproducibility is essential in machine learning for validating models, debugging issues, and comparing different runs or experiments consistently.

Firstly, let's understand why reproducibility matters:

Validation: When you need to verify that a model’s behavior is consistent under repeated trials.
Debugging: Helps identify if the same issue reoccurs with identical random data generation.
Collaboration: Ensures that colleagues or stakeholders can replicate results, fostering trust in findings.

Now, let's see how to set and use a random seed with np.random:

Import the NumPy library: import numpy as np.
Set the seed using np.random.seed(seed_value) where seed_value is an integer representing the desired seed.
Any operation involving randomness will now yield consistent results.

Let's walk through a practical example:

PYTHON
# Step 1: Import NumPy
import numpy as np

# Step 2: Set the random seed
np.random.seed(42)

# Step 3: Generate some random data to illustrate reproducibility
random_numbers = np.random.rand(5)
print("Random numbers with a fixed seed:", random_numbers)

# Repeat the same operation; should yield identical results due to the fixed seed
np.random.seed(42)
new_random_numbers = np.random.rand(5)
print("New random numbers with the same seed:", new_random_numbers)

In this example, both random_numbers and new_random_numbers will be exactly the same because we used the same seed value (42). This is a basic illustration of how setting seeds can make your experiments reproducible.

#### Common Mistakes
and Follow-Ups
Not Setting Seeds Properly: Forgetting to set a seed or using different seeds across runs.
Mixing Randomness Sources: Using multiple random number generators without ensuring they are synchronized (if necessary).
Overlooking Non-Random Operations: Some operations, like shuffling data, also require setting the seed if you want them to be reproducible.
Additional Considerations
For more complex setups involving parallel processes or distributed systems, consider using np.random.default_rng() which supports seeding in a way that is suitable for parallel execution.
If working with PyTorch, TensorFlow, or other deep learning frameworks, ensure their random number generators are also seeded properly to maintain reproducibility.

**Code:**

```python
import numpy as np

# Set the seed and generate random numbers
np.random.seed(42)
random_numbers = np.random.rand(5)
print("Random numbers with a fixed seed:", random_numbers)

# Repeat the same operation with the same seed to verify reproducibility
np.random.seed(42)
new_random_numbers = np.random.rand(5)
print("New random numbers with the same seed:", new_random_numbers)
```

**Example input / output:**

```text
Random numbers with a fixed seed: [0.37454012 0.95071431 0.73199394 0.59865715 0.15601864]
New random numbers with the same seed: [0.37454012 0.95071431 0.73199394 0.59865715 0.15601864]
```

**Flow:**

1. Rendering diagram…

