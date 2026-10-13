---
type: "practice"
difficulty: "Intermediate"
---

# In OpenCV, why does cv2.imread load images in BGR order and why can this cause bugs with matplotlib?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Input Resolution Trade-offs](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Input_Resolution_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with computer vision tasks using libraries like OpenCV and plotting tools such as Matplotlib, understanding how image data is stored and visualized is crucial. In OpenCV, images are loaded in BGR (Blue-Green-Red) order by default, which can lead to bugs when you try to plot the same image using Matplotlib, which expects RGB (Red-Green-Blue) format.

This question delves into both the technical reason behind BGR ordering and potential pitfalls it might cause. Let's break down the solution from first principles.

