---
type: "practice"
difficulty: "Intermediate"
---

# How would you build a reproducible image augmentation pipeline (e.g., Albumentations) and ensure augmentations applied at train time don't leak into the eval set?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Fine-tuning Pretrained CNNs](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Fine-tuning_Pretrained_CNNs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a reproducible image augmentation pipeline using a framework like Albumentations while ensuring that no augmentations are applied to the evaluation dataset, you need to carefully manage your data flow and ensure separation between training and validation phases. Here's how I would approach this:

First, the key idea is to use different configurations for the train and eval datasets to prevent any accidental leakage of augmentation parameters from one set to another. This involves setting up distinct pipelines within Albumentations that are applied only during training and not when evaluating the model.

Now let's break it down step by step:

