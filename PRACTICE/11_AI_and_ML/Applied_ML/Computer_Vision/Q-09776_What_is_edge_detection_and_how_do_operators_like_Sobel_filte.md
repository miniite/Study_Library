---
type: "practice"
difficulty: "Intermediate"
---

# What is edge detection and how do operators like Sobel filters find edges in an image?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Classification vs Detection](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Classification_vs_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Edge detection is a fundamental concept in computer vision, used to identify points in a digital image at which the intensity changes sharply or has discontinuities. The goal of edge detection is to identify and mark transitions between different regions in an image, such as the boundaries of objects. Sobel filters are among the most popular operators for detecting edges by measuring the gradient magnitude of the image.

In this answer, we will explore what edge detection is, how it works, and specifically dive into the Sobel filter technique.

**Comparison:**

```
Pixel	Gradients (X)	Gradients (Y)	Magnitude
1	-3	5	5.83
2	0	-4	4
3	2	-2	2.83
```

**Code:**

```python
# Example Python code to apply a Sobel filter using OpenCV
import cv2
import numpy as np

def sobel_filter(image_path):
    # Load an image in grayscale mode
    img = cv2.imread(image_path, 0)

    # Apply Sobel filters for gradient calculation along X and Y axes
    sobelx = cv2.Sobel(img, ddepth=cv2.CV_64F, dx=1, dy=0, ksize=3)
    sobely = cv2.Sobel(img, ddepth=cv2.CV_64F, dx=0, dy=1, ksize=3)

    # Calculate gradient magnitude and direction
    gradmag = np.sqrt(sobelx**2 + sobely**2)
    scale_factor = 1.0 / np.max(gradmag)
    gradmag = (gradmag * scale_factor).astype(np.uint8)

    return gradmag
```

**Example input / output:**

```

Input image path:

JSON
Copy
"images/example_image.jpg"

Output after applying Sobel filter (gradient magnitude):

PYTHON
Copy
output_image = sobel_filter("images/example_image.jpg")
cv2.imshow('Sobel Filtered Image', output_image)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

**Data:**

```json
{
  "image_path": "path/to/your/image.png"
}
```

**Flow:**

1. Yes
2. Yes
3. Load Image
4. Convert to Grayscale
5. Is the image in grayscale?
6. Apply Sobel Filter (Sobelx,Sobely)
7. Compute Gradient Magnitude
8. Scale and Normalize
9. Output Image

