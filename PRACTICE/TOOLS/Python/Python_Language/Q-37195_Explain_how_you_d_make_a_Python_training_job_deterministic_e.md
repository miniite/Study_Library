---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd make a Python training job deterministic end-to-end (data order, weight init, dropout) so two runs with the same seed produce identical models, and why full determinism is hard on GPU.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Drift-triggered Retraining](<../../../../TOOLS/Python/Python_Language/Python_Tail_Latency/Drift-triggered_Retraining.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a machine learning training job in Python produces deterministic results across multiple runs, we need to address several aspects of the training pipeline, including data shuffling, weight initialization, and dropout operations. Determinism is particularly challenging on GPUs due to their parallel architecture and non-deterministic behavior introduced by hardware and driver implementations.

Firstly, let's define what determinism means in this context: identical results produced from the same inputs with the same random seeds. This requires controlling every source of randomness that could influence the training process.

**Code:**

```python
import numpy as np
import torch

# Set random seeds for Numpy and PyTorch to ensure reproducibility
np.random.seed(42)
torch.manual_seed(42)

# Ensure deterministic behavior in PyTorch (CPU/GPU)
torch.use_deterministic_algorithms(True)

# Enable cudnn benchmarking (disable for determinism, but may be important for performance tuning)
torch.backends.cudnn.benchmark = False
```

**Flow:**

1. Set Numpy Random Seed
2. Initialize PyTorch
3. Set Deterministic Algorithmsto True
4. Disable cuDNN Benchmarkingfor Determinism

