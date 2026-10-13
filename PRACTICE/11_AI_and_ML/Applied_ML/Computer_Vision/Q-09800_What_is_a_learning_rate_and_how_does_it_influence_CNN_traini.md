---
type: "practice"
difficulty: "Basic"
---

# What is a learning rate and how does it influence CNN training convergence?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Fine-tuning Pretrained CNNs](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Fine-tuning_Pretrained_CNNs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Learning rate in the context of Convolutional Neural Networks (CNNs) is a hyperparameter that determines the step size at each iteration while moving toward a minimum of the loss function during training. A smaller learning rate can help avoid overshooting the minimum, but might make the training process too slow; whereas a larger learning rate speeds up convergence but risks missing the optimal solution or even diverging.

To understand how the learning rate influences CNN training convergence, let's break it down from first principles.

