---
type: "practice"
difficulty: "Intermediate"
---

# Your training crashes with OOM on large images. Walk me through the levers (batch size, gradient accumulation, resolution, checkpointing) to fit memory.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Input Resolution Trade-offs](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Input_Resolution_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you encounter out-of-memory (OOM) errors during training in computer vision tasks, it's essential to manage resources effectively. The key levers are batch size, gradient accumulation, image resolution, and checkpointing. By understanding each of these components and their interactions, we can optimize the model's memory usage without compromising performance.

#### Triage
What’s Being Probed

The question is essentially asking for a detailed explanation of how to manage memory constraints in training large models on high-resolution images. The interviewer might be probing your knowledge of optimization techniques, understanding of trade-offs between these levers, and your ability to make strategic decisions based on the specific requirements of the task.

#### Assumptions
We are dealing with a deep learning model trained using PyTorch or TensorFlow.
The dataset consists of high-resolution images that require significant memory for processing.
We need to ensure efficient use of GPU memory while maintaining acceptable training performance.
Step-by-Step Solution
1. Batch Size

Initial Batch Size The batch size is the number of samples processed before the model’s internal parameters are updated during training. A smaller batch size means each update uses less memory, but it can slow down convergence and make the learning process noisy due to higher variance in the updates.

Optimization Strategy To fit the model into memory:

Reduce Batch Size: Start by reducing the batch size until you no longer encounter OOM errors.
Evaluate Performance: Monitor training metrics (accuracy, loss) to ensure performance is not adversely affected. If necessary, fine-tune the batch size incrementally.
2. Gradient Accumulation

Concept Gradient accumulation allows us to break down the gradient update process into multiple small updates over several forward-backward passes. This technique effectively increases the effective batch size without significantly increasing memory usage per pass.

#### Implementation
Strategy

Enable Gradient Accumulation: Implement a mechanism where gradients are accumulated in memory across multiple mini-batches, and then applied at once after a fixed number of iterations.
Calculate Adjusted Batch Size: The effective batch size becomes the product of the actual batch size and the number of accumulation steps.
3. Image Resolution

Initial Consideration Reducing image resolution can significantly reduce memory requirements, but it might also affect model performance if spatial details are crucial for the task.

Optimization Strategy

Resize Images: Try reducing the image dimensions while maintaining a balance between memory and performance.
Evaluate Impact: Perform a dry run with different resolutions to see how they impact training speed and accuracy.
4. Checkpointing

Concept Checkpointing involves saving the model's state periodically during training, which can be crucial for resuming training after an interruption without losing progress.

#### Implementation
Strategy

Save Checkpoints: Introduce a checkpointing mechanism where the model’s state is saved at regular intervals.
Resume Training: Use these checkpoints to resume training from the last saved state if needed.

#### Closing Summary

By strategically adjusting batch size, utilizing gradient accumulation, considering image resolution, and implementing effective checkpointing, we can optimize memory usage without compromising the training process. Each of these levers has its own trade-offs, so it's important to carefully evaluate and balance them based on the specific requirements of your task.

#### Common Mistakes/Follow-Ups
Mistake 1: Ignoring image resolution as a factor for reducing memory consumption.

#### Follow-Up
1: Discuss how downsampling images can be balanced against model performance.
Mistake 2: Not considering the impact on training speed when adjusting batch size or accumulation steps.

#### Follow-Up
2: Explain how to monitor and optimize trade-offs between speed and memory usage.

**Comparison:**

```
Leverage	Description	Adjusted Batch Size
Batch Size	Number of samples per update	16
Gradient Accumulation	Accumulate gradients over multiple passes	Effective batch size 64
Image Resolution	Reduce dimensions to fit memory constraints	960x540
```

**Code:**

```python
import torch

# Example of enabling gradient accumulation
def train_model(model, data_loader, optimizer, device='cuda'):
    model.train()
    for i, (images, labels) in enumerate(data_loader):
        images = images.to(device)
        labels = labels.to(device)

        # Zero the parameter gradients
        optimizer.zero_grad()

        outputs = model(images)
        loss = criterion(outputs, labels)

        # Accumulate gradients over multiple batches
        if i % 4 == 0:  # Adjust the accumulation step based on your needs
            loss.backward()
            optimizer.step()
```

**Example input / output:**

```text
Initial batch size: 32 -> OOM error.
Reduced batch size: 16 -> No OOM error but slower training.
Gradient accumulation steps: 4 -> Effective batch size 64, no OOM and acceptable performance.

Dry run:
- Original image resolution: 1920x1080
- Reduced image resolution: 960x540
- Training time with reduced resolution: 30% faster

Checkpoint interval: Every 10 epochs -> Resumes training smoothly after interruptions.
```

**Data:**

```json
{
    "batch_size": 8,
    "image_resolution": {"width": 1920, "height": 1080},
    "accumulation_steps": 4
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Yes
5. No
6. Start
7. Is OOM?
8. Reduce Batch Size
9. Enable GradientAccumulation?
10. Adjust Accumulation Steps
11. Reduce Image Resolution
12. Save Checkpoints?
13. Resume Training fromCheckpoint
14. Continue Training withAdjusted Parameters

