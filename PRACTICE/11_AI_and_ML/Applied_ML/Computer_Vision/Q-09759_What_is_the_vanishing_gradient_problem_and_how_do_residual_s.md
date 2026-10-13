---
type: "practice"
difficulty: "Deep"
---

# What is the vanishing gradient problem and how do residual (skip) connections in ResNet help address it?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Fine-tuning Pretrained CNNs](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Fine-tuning_Pretrained_CNNs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The vanishing gradient problem occurs during training deep neural networks, where gradients become very small as they propagate through layers, making learning difficult or impossible for deeper layers. In the context of computer vision, this means that very deep convolutional neural networks (CNNs) struggle to effectively learn and optimize parameters in their early layers.

Residual (skip) connections in ResNet address this issue by allowing information from earlier layers to bypass some layers and directly contribute to later layers. This helps maintain gradient flow through the network, making it easier for the model to train deeper architectures.

Let's dive into the first-principles solution step-by-step.

