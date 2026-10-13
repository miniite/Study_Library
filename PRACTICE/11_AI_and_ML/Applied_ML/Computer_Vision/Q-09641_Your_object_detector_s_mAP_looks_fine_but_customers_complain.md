---
type: "practice"
difficulty: "Deep"
---

# Your object detector's mAP looks fine but customers complain about missed small objects. How would you break down mAP by object size to confirm and then fix it?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Classification Metrics](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Classification_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we need to first understand what mAP (mean Average Precision) represents and how it is calculated. mAP is a common metric in object detection tasks that measures the average precision across different recall levels. However, as customers are complaining about missed small objects, breaking down the mAP by object size will help us identify where the problem lies. This step-by-step solution involves analyzing the performance on various object sizes and then refining the model accordingly.

#### Triage
What the Question is Asking

The question requires a detailed breakdown of the mAP metric to understand which object sizes are causing the model to miss objects, thereby leading to customer complaints. The goal is not only to confirm whether small objects are being missed but also to identify where in the model's architecture or training process this issue lies so that we can take corrective actions.

#### Assumptions
and Key Idea
We assume that mAP is calculated across all object sizes.
Small objects are defined as those below a certain size threshold, which needs to be established based on domain knowledge or predefined criteria.
The model's performance will be analyzed by re-evaluating the detection results in terms of different object size bins.
Step-by-Step Solution

Define Object Size Bins: First, define what constitutes small objects. For instance, if we are working with images of varying resolutions, a common approach might be to use thresholds such as 32x32, 64x64, and so on.

Re-evaluate mAP by Size Bins:

Split the dataset into different object size bins based on predefined criteria (e.g., area).
Calculate the mAP for each bin separately. This can be done using a tool like COCO eval or any other evaluation script that supports such customization.

Analyze Results:

If the overall mAP looks fine but the small objects have a significantly lower mAP, it indicates that the model is underperforming in detecting smaller objects.
Investigate further by looking at precision and recall separately within each size bin to understand which phase of the detection pipeline (e.g., feature extraction, localization, classification) might be failing.

Identify Root Causes:

Analyze the features extracted from small objects versus larger ones to see if there are significant differences in how these objects are represented.
Check for potential issues like overfitting on large objects and poor generalization to smaller ones, or vice versa.

Fix Identified Issues:

If feature extraction is a problem, consider using different pre-trained models or fine-tuning the model with more data specifically focusing on small objects.
For localization issues, experiment with anchor boxes or other bounding box generation techniques that work well for smaller objects.
If classification accuracy is an issue, look at adjusting class weights in the loss function to favor correctly identifying small objects.

#### Closing Summary
and Common Mistakes

By breaking down mAP by object size, we can pinpoint where the model's weaknesses lie and take targeted steps to improve its performance. A common mistake is assuming that overall mAP being good means there are no issues with detecting smaller objects; therefore, it’s crucial to conduct a thorough analysis.

#### Follow-Ups
How do you handle imbalanced classes in object detection?
What techniques can be used to enhance the model's ability to detect small objects?

**Comparison:**

```
Size Bin	Bounding Boxes (x_min, y_min, x_max, y_max)
0-32x32	[100, 200, 300, 400]
0-64x64	[50, 60, 70, 80]
32x32+	[90, 100, 110, 120]
```

**Code:**

```python
import numpy as np

def calculate_map_by_size(classes, bboxes, labels, size_thresholds):
    # Define size bins and corresponding thresholds
    size_bins = []
    for i in range(len(size_thresholds) - 1):
        if i == 0:
            size_bins.append((0, size_thresholds[i]))
        else:
            size_bins.append((size_thresholds[i-1], size_thresholds[i]))

    # Split the data by size bins
    split_data = {}
    for bbox, label in zip(bboxes, labels):
        area = bbox[2] * bbox[3]
        for i, (low, high) in enumerate(size_bins):
            if low <= area < high:
                if i not in split_data:
                    split_data[i] = {'bbox': [], 'label': []}
                split_data[i]['bbox'].append(bbox)
                split_data[i]['label'].append(label)
                break

    # Calculate mAP for each bin
    aps = []
    for key, data in split_data.items():
        bbox = np.array(data['bbox'])
        label = np.array(data['label'])
        ap = calculate_precision_recall(bbox, label)  # Placeholder function
        aps.append(ap)

    return np.mean(aps)

def calculate_precision_recall(bboxes, labels):
    # Calculate precision and recall for a specific class or all classes
    pass
```

**Example input / output:**

```text
size_thresholds = [32*32, 64*64]  # Define size bins

classes = np.array([1, 2, 3])
bboxes = np.array([
    [100, 200, 300, 400],  # Large object
    [50, 60, 70, 80],      # Small object
    [90, 100, 110, 120]   # Medium object
])
labels = np.array([1, 2, 3])

mAP_by_size = calculate_map_by_size(classes, bboxes, labels, size_thresholds)
print(f"mAP by size: {mAP_by_size}")
```

**Data:**

```json
{
  "bboxes": [
    [100, 200, 300, 400],  // Coordinates of the bounding box (x_min, y_min, x_max, y_max)
    [50, 60, 70, 80],
    [90, 100, 110, 120]
  ],
  "labels": [
    1,  # Class label
    2,
    3
  ]
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Define Object Size Bins
5. Split Data by Size?
6. Calculate mAP for Each Bin
7. Analyze Results?
8. Identify Root Causes
9. Try Different Techniques(Feature Extraction,Localization, Classification)
10. Fix Identified Issues

