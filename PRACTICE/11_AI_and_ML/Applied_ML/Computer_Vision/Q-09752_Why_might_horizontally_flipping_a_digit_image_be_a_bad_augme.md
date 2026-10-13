---
type: "practice"
difficulty: "Deep"
---

# Why might horizontally flipping a digit image be a bad augmentation but flipping a cat image be fine?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data vs Model Improvements](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Data_vs_Model_Improvements.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In machine learning, data augmentation is used to artificially expand a dataset by applying random transformations such as rotations, flips, and translations. The goal is to make models more robust and generalize better on unseen data. However, not all types of augmentations are equally beneficial for every type of image.

Horizontally flipping digit images can sometimes lead to poor performance because digits like "6" and "9" look fundamentally different when flipped horizontally, making it harder for the model to learn their distinguishing features. On the other hand, flipping cat images is generally acceptable because cats maintain recognizable features even after a horizontal flip—they still have whiskers, ears, and tails.

This situation highlights why understanding the content of your data is crucial in choosing appropriate augmentations.

