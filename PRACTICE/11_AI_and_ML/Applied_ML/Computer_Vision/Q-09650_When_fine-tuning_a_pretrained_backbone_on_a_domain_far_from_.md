---
type: "practice"
difficulty: "Deep"
---

# When fine-tuning a pretrained backbone on a domain far from ImageNet (e.g., medical X-rays), how do you decide how many layers to unfreeze and at what learning rates?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Fine-tuning Pretrained CNNs](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Fine-tuning_Pretrained_CNNs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Fine-tuning a pretrained backbone like ResNet or EfficientNet on medical X-ray images requires a nuanced approach due to the domain shift from ImageNet, which primarily contains natural images. The goal is to leverage pre-trained weights while adapting them to the specific characteristics of medical imaging data. To decide how many layers to unfreeze and at what learning rates, we need to balance model adaptation with preventing catastrophic forgetting.

#### Triage
What's Being Probed

The core question revolves around understanding the trade-offs in fine-tuning deep neural networks, specifically when dealing with a domain shift. The interviewer is probing your knowledge of:

The role of pre-trained models and transfer learning.
How to handle domain shifts in computer vision tasks.
Practical strategies for fine-tuning hyperparameters like unfreezing layers and setting learning rates.

#### Assumptions
We are working with a large pretrained backbone, such as ResNet or EfficientNet.
The dataset for the medical X-ray task is small relative to ImageNet.
The goal is to improve model performance on the new domain while maintaining generalization on broader image data.
Step-by-Step Solution

Understand the Domain Shift:

ImageNet vs Medical X-rays: ImageNet contains a vast variety of natural images, making it challenging for models to generalize to medical imaging tasks.
Layer-wise Transfer Learning: The deeper layers capture more generic features (like edges and textures), while shallower layers capture object-specific details.

Initial Unfreezing Strategy:

Start by unfreezing the top few layers of the pretrained model. This allows the network to adapt to the new domain without disrupting the learned, higher-level representations.
For example, in a ResNet-50, unfreeze the last 1 or 2 blocks.

Learning Rate Schedule:

Use a lower learning rate for the unfrozen layers and a much lower rate for the pretrained weights to avoid destabilizing them.

#### Example
learning_rate = 1e-4 for unfrozen layers, learning_rate = 5e-6 for pretrained layers.

Monitoring Training Dynamics:

Monitor training loss on both validation sets (one from ImageNet and one from the medical X-rays).
Use techniques like Grad-CAM or feature visualization to understand how different layers are responding to the new domain.

Iterative Refinement:

Gradually unfreeze more layers as needed, monitoring performance on a separate validation set specific to the new task.
Be cautious about overfitting by regularly checking generalization metrics.

Hyperparameter Tuning:

Perform grid search or random search for optimal hyperparameters within reasonable ranges.

#### Cons
ider using techniques like early stopping and learning rate annealing to further stabilize training.

#### Closing Summary

When fine-tuning a pretrained backbone on medical X-rays, start by unfreezing the top few layers with lower learning rates. Gradually increase the number of unfrozen layers based on model performance metrics. Monitoring training dynamics and iteratively refining the unfreeze strategy will help achieve good generalization to both ImageNet and the new domain.

#### Common Mistakes/Follow-Ups
Mistake: Unfreezing too many layers at once, leading to catastrophic forgetting.
Mistake: Using high learning rates for unfrozen layers, causing model divergence.

#### Follow-up
How would you handle an even larger pretrained model like a Vision Transformer (ViT)? Would the approach change if the dataset were significantly larger?

